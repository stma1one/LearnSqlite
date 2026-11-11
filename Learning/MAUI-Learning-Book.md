# ספר הלמידה וההדרכה: LearnSqlite — מארכיטקטורת MVVM ועד מסד נתונים SQLite

ברוכים הבאים לספר הלמידה המלווה את פרויקט **LearnSqlite** ב-.NET MAUI!
מטרת הפרויקט היא להוביל תלמידים צעד-אחר-צעד מבניית אפליקציית **MVVM** תלת-מסכית נקייה העובדת מול נתוני דמה בזיכרון (**Mockup Service** בענף `master`), ועד להטמעת מסד נתונים מקומי אמיתי מבוסס **SQLite** בענף ההמשך — מבלי לשבור את קוד המסכים או ה-ViewModels.

---

## 1. מפת הדרכים הפדגוגית של הפרויקט (Project Roadmap)

הפרויקט בנוי בשני שלבים מרכזיים (ענפי Git נפרדים), כך שהתלמיד חווה בעצמו את הכוח האמיתי של הפרדת שכבות וממשקים (`Interfaces`):

| שלב / ענף | תוצאה נראית לעין | קבצים מרכזיים | מושג מפתח |
| --- | --- | --- | --- |
| **שלב א' (`master`)** | אפליקציית 3 מסכים פעילה עם CRUD מלא בזיכרון | `HeroItem`, `IHeroService`, `MockHeroService`, 3 `ViewModels`, 3 `Views` | הפרדת שכבות, `INotifyPropertyChanged`, `ICommand`, ניווט `AppShell` |
| **שלב ב' (`sqlite`)** | שמירה קבועה בקובץ מסד נתונים מקומי במכשיר | `SqliteHeroService`, תכונות `[PrimaryKey]` במודל | `SQLiteAsyncConnection`, מיפוי ORM, החלפת שירות שקופה |

> [!IMPORTANT]
> ### למה מתחילים עם `MockHeroService` לפני שמתקינים SQLite?
> כאשר מפרידים בין **ממשק השירות (`IHeroService`)** לבין **המימוש שלו (`MockHeroService`)**, התלמיד בונה ובודק את כל מסכי האפליקציה, הטפסים, הפקודות והניווט בסביבה פשוטה ומהירה. לאחר שהאפליקציה עובדת ב-100%, המעבר ל-SQLite מתמקד אך ורק בשכבת הנתונים!

---

## 2. ארכיטקטורת השכבות בפרויקט הבסיס (`master`)

פרויקט הבסיס בנוי בארכיטקטורת **MVVM** מוקפדת, ללא הזרקת תלויות (DI) בשלב זה, כדי לשמור על שקיפות מלאה של זרימת האובייקטים עבור התלמיד:

```text
LearnSqlite/
├── Models/
│   └── HeroItem.cs                # מודל הנתונים (POCO) של גיבור-על
├── Services/
│   ├── IHeroService.cs            # חוזה א-סינכרוני לכל פעולות ה-CRUD
│   └── MockHeroService.cs         # מימוש בזיכרון עם 6 גיבורי-על ותמונות חיות
├── ViewModels/
│   ├── BaseViewModel.cs           # מימוש INotifyPropertyChanged ומצב טעינה
│   ├── HeroesListViewModel.cs     # לוגיקת מסך האוסף, חיפוש וניווט
│   ├── AddHeroViewModel.cs        # לוגיקת מסך הוספת גיבור ואימות טופס
│   └── HeroDetailsViewModel.cs    # לוגיקת מסך פרטים, עדכון ומחיקה (IQueryAttributable)
├── Views/
│   ├── HeroesListPage.xaml(.cs)   # מסך 1: צפייה באוסף וחיפוש
│   ├── AddHeroPage.xaml(.cs)      # מסך 2: טופס הוספת רשומה חדשה
│   └── HeroDetailsPage.xaml(.cs)  # מסך 3: צפייה בפרטי רשומה, עדכון ומחיקה
└── AppShell.xaml(.cs)             # ניהול מעטפת האפליקציה ורישום נתיבי ניווט (Routes)
```

---

## 3. צעד 1: מודל הנתונים — `Models/HeroItem.cs`

### מטרת הקובץ ומקומו בארכיטקטורה
המודל מייצג רשומה בודדת של גיבור-על בארכיון. בחרנו במודל עשיר הכולל טיפוסי נתונים מגוונים (`int`, `string`, `bool`) כדי להכין את הקרקע למיפוי עמודות בטבלת SQLite בהמשך. כל תמונות הגיבורים נטענות מכתובות URL מאומתות ברשת (מתוך מאגר ה-CDN הציבורי `akabab/superhero-api`).

```csharp
namespace LearnSqlite.Models;

public class HeroItem
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public string AlterEgo { get; set; } = string.Empty;
    public string Publisher { get; set; } = "Marvel";
    public int PowerLevel { get; set; } = 75;
    public bool IsEliteMember { get; set; }
    public string ImageUrl { get; set; } = string.Empty;

    // מאפייני עזר לקריאה בלבד עבור תצוגה נקייה ב-XAML
    public string PowerBadgeText => $"⚡ עוצמה: {PowerLevel}/100";
    public string EliteStatusText => IsEliteMember ? "⭐ חבר צוות עלית" : "🛡️ פועל עצמאית";
}
```

| שם המאפיין | טיפוס ב-C# | תפקיד עתידי ב-SQLite |
| --- | --- | --- |
| `Id` | `int` | מפתח ראשי מספור אוטומטי (`[PrimaryKey, AutoIncrement]`) |
| `Name` / `AlterEgo` | `string` | עמודות טקסט (`TEXT`) לחיפוש וסינון |
| `Publisher` | `string` | עמודת קטגוריה (`Marvel` / `DC` / `Independent`) |
| `PowerLevel` | `int` | עמודת מספר שלם (`INTEGER`) בטווח 1–100 |
| `IsEliteMember` | `bool` | עמודה בוליאנית (`0` או `1` ב-SQLite) |
| `ImageUrl` | `string` | כתובת URL לתמונת הגיבור המוצגת בפקד `Image` |

---

## 4. צעד 2: חוזה ה-CRUD ומימוש ה-Mockup — `Services/`

### 4.1 ממשק השירות: `Services/IHeroService.cs`
כל פעולות השירות מוגדרות כא-סינכרוניות (`Task`) ומקבילות בדיוק לפעולות מסד הנתונים שנממש בהמשך עם `SQLiteAsyncConnection`:

```csharp
using LearnSqlite.Models;

namespace LearnSqlite.Services;

public interface IHeroService
{
    Task<List<HeroItem>> GetAllHeroesAsync();
    Task<HeroItem?> GetHeroByIdAsync(int id);
    Task<List<HeroItem>> SearchHeroesAsync(string searchText);
    Task<int> AddHeroAsync(HeroItem hero);
    Task<int> UpdateHeroAsync(HeroItem hero);
    Task<int> DeleteHeroAsync(int id);
}
```

> [!TIP]
> ### 💡 מדוע `AddHeroAsync`, `UpdateHeroAsync` ו-`DeleteHeroAsync` מחזירות `Task<int>`?
> בספריית `sqlite-net-pcl`, מתודות ה-CRUD (`InsertAsync`, `UpdateAsync`, `DeleteAsync`) מחזירות `Task<int>` המייצג את **מספר השורות שהושפעו בטבלה** (`Rows Affected`). הגדרת החתימה הזו כבר בממשק ה-Mockup מבטיחה תאימות של 100% במעבר ל-SQLite!

### 4.2 מימוש הדמה בזיכרון: `Services/MockHeroService.cs`
כיוון שבשלב זה איננו משתמשים ב-Dependency Injection (DI), כל מסך יוצר מופע של `new MockHeroService()`. כדי שכל 3 המסכים יקראו ויעדכנו את **אותו אוסף רשומות**, הרשימה `_heroes` והמונה `_nextId` מוגדרים כ-`static`:

```csharp
public class MockHeroService : IHeroService
{
    private static readonly List<HeroItem> _heroes = new()
    {
        new HeroItem
        {
            Id = 1,
            Name = "באטמן (Batman)",
            AlterEgo = "ברוס ויין",
            Publisher = "DC",
            PowerLevel = 88,
            IsEliteMember = true,
            ImageUrl = "https://raw.githubusercontent.com/akabab/superhero-api/0.3.0/api/images/md/70-batman.jpg"
        },
        // ... 5 גיבורים נוספים עם כתובות תמונה מאומתות
    };

    private static int _nextId = 7;

    public async Task<int> AddHeroAsync(HeroItem hero)
    {
        await Task.Delay(150);
        var copy = CloneHero(hero);
        copy.Id = _nextId++;
        _heroes.Add(copy);
        hero.Id = copy.Id;
        return 1;
    }
}
```

> [!NOTE]
> ### מדוע אנו מבצעים `CloneHero` בשליפה ובשמירה?
> במסד נתונים אמיתי (כמו SQLite), כאשר שולפים רשומה למסך העריכה ומשנים שדה ב-ViewModel, הרשומה בטבלה **אינה משתנה** עד שלוחצים על כפתור "עדכן רשומה" (`UpdateHeroAsync`). שכפול האובייקט (`CloneHero`) ב-`MockHeroService` מחקה בדיוק את ההתנהגות של מסד נתונים אמיתי, כך שאם המשתמש משנה טקסט ולוחץ "חזרה ללא שמירה", הנתון המקורי ברשימה לא נפגע!

---

## 5. צעד 3: מחלקת הבסיס `ViewModels/BaseViewModel.cs`

מחלקת `BaseViewModel` מממשת את `INotifyPropertyChanged` עם מאפיינים מפורשים (`get`/`set` מלאים ללא קיצורים חבויים) ומספקת ניהול אחיד של:
- `IsLoading` ו-`IsNotLoading` (עבור פקד `ActivityIndicator` ותנאי `CanExecute`).
- `Title` (כותרת המסך).
- `StatusMessage`, `HasStatusMessage` ו-`HasError` (להצגת משוב ושגיאות למשתמש).
- מתודה וירטואלית `OnLoadingStateChanged()` המאפשרת לכל ViewModel יורש לרענן אוטומטית את ה-`CanExecute` של הפקודות שלו ברגע שמתחילה או מסתיימת טעינה.

```csharp
public bool IsLoading
{
    get => _isLoading;
    set
    {
        if (_isLoading != value)
        {
            _isLoading = value;
            OnPropertyChanged();
            OnPropertyChanged(nameof(IsNotLoading));
            OnLoadingStateChanged();
        }
    }
}
```

---

## 6. צעד 4: ניווט רב-מסכי באמצעות `AppShell`

האפליקציה כוללת 3 מסכים המנוהלים על ידי `AppShell`:
1. **`HeroesListPage` (מסך הבית הראשי):** מוגדר כ-`ShellContent` בתוך `AppShell.xaml` עם `FlowDirection="RightToLeft"`.
2. **`AddHeroPage` (מסך הוספה):** נרשם כנתיב ניווט (`Routing.RegisterRoute`) ב-`AppShell.xaml.cs`.
3. **`HeroDetailsPage` (מסך פרטי/עדכון רשומה):** נרשם כנתיב ניווט ב-`AppShell.xaml.cs` ומקבל את מזהה הרשומה (`HeroId`) באמצעות `Dictionary<string, object>` והממשק `IQueryAttributable`.

```csharp
public partial class AppShell : Shell
{
    public AppShell()
    {
        InitializeComponent();

        Routing.RegisterRoute(nameof(AddHeroPage), typeof(AddHeroPage));
        Routing.RegisterRoute(nameof(HeroDetailsPage), typeof(HeroDetailsPage));
    }
}
```

---

## 7. אינדקס מדריכי המסכים (שיטת הבלוקים המודולרית)

בהמשך ספר זה מופיעים 3 מדריכי פיתוח מפורטים (פרק לכל מסך), הכוללים הדמיות חזותיות, חלוקה לבלוקים ב-XAML וב-ViewModel, ונקודות בקרה לתלמיד:

1. **מדריך מסך 1 (`01-HeroesListPage-Guide.md`):** מסך הצפייה באוסף, חיפוש וניווט (`HeroesListPage` + `HeroesListViewModel`).
2. **מדריך מסך 2 (`02-AddHeroPage-Guide.md`):** מסך הוספת גיבור חדש ואימות קלט (`AddHeroPage` + `AddHeroViewModel`).
3. **מדריך מסך 3 (`03-HeroDetailsPage-Guide.md`):** מסך פרטי גיבור, קבלת `HeroId` בניווט, עדכון ומחיקה (`HeroDetailsPage` + `HeroDetailsViewModel`).
4. **מדריך שלב 4 (`04-SqliteDatabase-Guide.md`):** חיבור מסד הנתונים המקומי SQLite (`SqliteHeroService`) בענף `sqlite`.

---

## 8. שלב ב' (ענף `sqlite`): מעבר מ-`MockHeroService` למסד נתונים מקומי `SQLite`

לאחר שווידאנו שכל 3 המסכים וזרימת ה-MVVM פועלים כשורה בענף `master`, יצרנו את ענף `sqlite` וביצענו בו 4 צעדים ממוקדים:

### 8.1 התקנת חבילת ה-NuGet של SQLite ב-`LearnSqlite.csproj`
* **`sqlite-net-pcl` (גרסה `1.11.285`):** החבילה הישירה היחידה שנדרשת בפרויקט הדוגמה עבור CRUD מול SQLite, באמצעות `SQLiteAsyncConnection`.
* אין להוסיף הפניה ישירה ל-`SQLitePCLRaw.bundle_green` בהתקנה הרגילה; רכיבי SQLite native נפתרים דרך גרף התלויות של `sqlite-net-pcl`. לאחר Restore יש לבדוק את התלויות והחולשות הטרנזיטיביות.
* **`SQLiteNetExtensions` אינה מותקנת בפרויקט הזה**, משום שיש בו טבלה אחת ללא קשרים. בפרויקט שדורש one-to-one, one-to-many או many-to-many מוסיפים אותה רק לאחר בדיקת התאימות וה-API, לפי [מדריך one-to-one](https://www.codejourney.net/sqlite-net-extensions-one-to-one-relationships/), [מדריך one-to-many](https://www.codejourney.net/sqlite-net-extensions-one-to-many-relationships/) ו-[מדריך many-to-many](https://www.codejourney.net/sqlite-net-extensions-many-to-many-relationships/).

בדיקות מומלצות לאחר התקנה:
```bash
dotnet restore
dotnet build
dotnet list LearnSqlite/LearnSqlite.csproj package --vulnerable --include-transitive
```

### 8.2 עיטור המודל `HeroItem` בתכונות מיפוי לטבלה (Attributes)
במחלקה `LearnSqlite/Models/HeroItem.cs` הוספנו את תכונות ה-ORM של `SQLite`:
* `[Table("Heroes")]` — מגדיר את שם הטבלה במסד הנתונים.
* `[PrimaryKey, AutoIncrement]` מעל `Id` — קובע מפתח ראשי עם מספור אוטומטי ב-SQLite.
* `[MaxLength(100), NotNull]` מעל `Name` — מגדיר אילוץ שדה חובה ואורך מקסימלי.
* `[Ignore]` מעל `PowerBadgeText` ו-`EliteStatusText` — מונע יצירת עמודות בטבלה עבור מאפייני עזר מחושבים לתצוגה.

### 8.3 מימוש `SqliteHeroService` עם אתחול עצל (`InitAsync`) וזריעת נתונים (Seeding)
במחלקה `LearnSqlite/Services/SqliteHeroService.cs` מימשנו את החוזה `IHeroService` מול הקובץ המקומי `Path.Combine(FileSystem.AppDataDirectory, "HeroesArchive.db3")`:
* **בנאי נקי:** בהתאם לכללי ה-Guardrails של .NET MAUI, הבנאי `SqliteHeroService()` אינו מבצע פעולות קובץ או מסד נתונים.
* **אתחול א-סינכרוני עצל (`InitAsync`):** נקרא בתחילת כל פעולת CRUD, מוגן על ידי `SemaphoreSlim`, יוצר את הטבלה (`CreateTableAsync<HeroItem>()`) ובפעם הראשונה שבה הטבלה ריקה (`CountAsync() == 0`) זורע אוטומטית את 6 גיבורי-העל ההתחלתיים באמצעות `InsertAllAsync`.
* **פעולות ה-CRUD:**
  * `GetAllHeroesAsync()` -> `Table<HeroItem>().OrderByDescending(h => h.PowerLevel).ToListAsync()`
  * `GetHeroByIdAsync(int id)` -> `Table<HeroItem>().Where(h => h.Id == id).FirstOrDefaultAsync()`
  * `SearchHeroesAsync(string searchText)` -> סינון א-סינכרוני לפי שם, זהות סודית או יקום קומיקס
  * `AddHeroAsync(HeroItem hero)` -> `InsertAsync(hero)`
  * `UpdateHeroAsync(HeroItem hero)` -> `UpdateAsync(hero)`
  * `DeleteHeroAsync(int id)` -> `DeleteAsync<HeroItem>(id)`

### 8.4 החלפת השירות ב-Views ללא שינוי ב-ViewModels או ב-XAML
בזכות העובדה שכל שלושת ה-ViewModels תלויים בממשק `IHeroService` בלבד, המעבר מנתוני דמה בזיכרון למסד נתונים SQLite דרש החלפה של שם המחלקה בלבד ב-Code-Behind של שלושת המסכים (`new SqliteHeroService()` במקום `new MockHeroService()`), ללא שינוי של אף שורת קוד ב-ViewModels או ב-XAML!

