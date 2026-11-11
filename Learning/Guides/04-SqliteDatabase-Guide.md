# מדריך שלב 4: חיבור מסד נתונים מקומי SQLite (`SqliteHeroService`)

## 1. מטרת השלב והארכיטקטורה

בשלושת השלבים הראשונים (ענף `master`) בנינו את האפליקציה בשלמותה מול שירות דמה בזיכרון (`MockHeroService`) המממש את הממשק `IHeroService`.  
בשלב זה (ענף `sqlite`) אנו מחליפים את שכבת הנתונים במסד נתונים מקומי אמיתי מסוג **SQLite** השומר את הנתונים בקובץ קבוע (`HeroesArchive.db3`) על גבי המכשיר — כך שכל הוספה, עריכה או מחיקה נשמרות גם לאחר סגירת האפליקציה והפעלתה מחדש.

> **מודל ויזואלי של ארכיטקטורת השלב (הדמיית HTML לימודית של 4 הבלוקים במעבר ל-SQLite):**

![סקירת ארכיטקטורת המעבר ל-SQLite](../Images/SqliteDatabase-overview.png)

---

## 2. פירוק לבלוקים (Step-by-Step Blocks)

### בלוק 1: התקנת חבילות ה-NuGet עבור SQLite ב-.NET MAUI

* **קובץ ושכבה:** `LearnSqlite/LearnSqlite.csproj` (הגדרות הפרויקט).
* **מה הקוד עושה:** מוסיף חבילת NuGet ישירה אחת בלבד:
  - `sqlite-net-pcl` (בפרויקט הדוגמה: גרסה `1.11.285`) — ספריית SQLite-net למיפוי מחלקות C# לטבלאות ולפעולות CRUD א-סינכרוניות באמצעות `SQLiteAsyncConnection`.

החבילה מביאה את רכיבי SQLite הנדרשים דרך גרף התלויות שלה. אין להוסיף לפרויקט הפניה ישירה נפרדת ל-`SQLitePCLRaw.bundle_green` בהתקנה הרגילה של `sqlite-net-pcl`. יש לבדוק את גרף התלויות בפועל אחרי Restore.

```xml
<ItemGroup>
    <PackageReference Include="Microsoft.Maui.Controls" Version="$(MauiVersion)" />
    <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="10.0.0" />
    <PackageReference Include="sqlite-net-pcl" Version="1.11.285" />
</ItemGroup>
```

**התקנה דרך CLI:**
```bash
dotnet add LearnSqlite/LearnSqlite.csproj package sqlite-net-pcl --version 1.11.285
dotnet restore
dotnet build
dotnet list LearnSqlite/LearnSqlite.csproj package --vulnerable --include-transitive
```

> **הערת גרסאות ואבטחה:** הדוגמה עודכנה מגרסה `1.9.172` לגרסה יציבה חדשה יותר `1.11.285`. בגרסה הישנה דווחה חולשת אבטחה ברכיב SQLite native שנמשך כתלות; לכן אין להעתיק גרסאות ישנות או לקבע ידנית bundle ישן. יש לבדוק את NuGet ואת התלויות הטרנזיטיביות בכל עדכון.

* **נקודת בדיקה לתלמיד (Checkpoint):** איזו חבילה מוסיפים ישירות לפרויקט, ואיך אפשר לבדוק אילו רכיבים נוספים הותקנו כתלויות שלה?

---

### מתי נדרשת חבילת קשרים?

בפרויקט הדוגמה הנוכחי יש טבלת `Heroes` אחת בלבד, ולכן **אין להתקין** `SQLiteNetExtensions`.

בפרויקט אחר, אם המודל באמת כולל קשרים בין טבלאות, אפשר להוסיף את NuGet `SQLiteNetExtensions` בנוסף ל-`sqlite-net-pcl`:
- [One-to-one](https://www.codejourney.net/sqlite-net-extensions-one-to-one-relationships/)
- [One-to-many](https://www.codejourney.net/sqlite-net-extensions-one-to-many-relationships/)
- [Many-to-many](https://www.codejourney.net/sqlite-net-extensions-many-to-many-relationships/)

יש לוודא את גרסת החבילה, תאימותה לגרסת `sqlite-net-pcl`, ואת שמות ה-attributes ומתודות הטעינה/שמירה בתיעוד לפני השימוש. אין להוסיף ORM או ספק SQLite אחר.

### בלוק 2: עיטור המודל `HeroItem` בתכונות מיפוי לטבלה (SQLite Attributes)

![בלוק 2 - מיפוי המודל HeroItem לטבלת SQLite](../Images/SqliteDatabase-block2-table-model.png)

* **קובץ ושכבה:** `LearnSqlite/Models/HeroItem.cs` (שכבת ה-Model).
* **מה הקוד עושה:** מוסיף תכונות (Attributes) מתוך מרחב השמות `SQLite` מעל המחלקה והמאפיינים שלה:
  * `[Table("Heroes")]` — קובע את שם הטבלה במסד הנתונים.
  * `[PrimaryKey, AutoIncrement]` מעל `Id` — מגדיר את `Id` כמפתח ראשי שמקבל מספור אוטומטי מ-SQLite בכל `InsertAsync`.
  * `[MaxLength(100), NotNull]` מעל `Name` — מגדיר אילוץ אורך וחובה ברמת הטבלה.
  * `[Ignore]` מעל `PowerBadgeText` ו-`EliteStatusText` — מורה ל-SQLite לא ליצור עמודות עבור מאפייני תצוגה מחושבים.

```csharp
using SQLite;

namespace LearnSqlite.Models;

[Table("Heroes")]
public class HeroItem
{
    [PrimaryKey, AutoIncrement]
    public int Id { get; set; }

    [MaxLength(100), NotNull]
    public string Name { get; set; } = string.Empty;

    [MaxLength(100)]
    public string AlterEgo { get; set; } = string.Empty;

    [MaxLength(50)]
    public string Publisher { get; set; } = "Marvel";

    public int PowerLevel { get; set; } = 75;

    public bool IsEliteMember { get; set; }

    public string ImageUrl { get; set; } = string.Empty;

    [Ignore]
    public string PowerBadgeText => $"⚡ עוצמה: {PowerLevel}/100";

    [Ignore]
    public string EliteStatusText => IsEliteMember ? "⭐ חבר צוות עלית" : "🛡️ פועל עצמאית";
}
```

* **נקודת בדיקה לתלמיד (Checkpoint):** מה יקרה בעת קריאה ל-`InsertAsync(newHero)` אם נשכח להוסיף `[AutoIncrement]` מעל המאפיין `Id`?

---

### בלוק 3: מימוש `SqliteHeroService` — אתחול עצל (`InitAsync`), זריעת נתונים ופעולות CRUD

![בלוק 3 - מימוש SqliteHeroService ופעולות CRUD](../Images/SqliteDatabase-block3-lazy-init-crud.png)

* **קובץ ושכבה:** `LearnSqlite/Services/SqliteHeroService.cs` (שכבת ה-Service).
* **מה הקוד עושה:**
  1. מחשב את נתיב קובץ מסד הנתונים באמצעות `Path.Combine(FileSystem.AppDataDirectory, "HeroesArchive.db3")`.
  2. שומר על כלל ה-Guardrail החשוב: **הבנאי (`SqliteHeroService()`) ריק ואינו מבצע פעולות קובץ או מסד נתונים**.
  3. מפעיל מתודת אתחול א-סינכרונית עצלה `InitAsync()` בתחילת כל פעולת CRUD, המוגנת באמצעות `SemaphoreSlim` מפני מצבי מרוץ (Race Conditions), יוצרת את הטבלה (`CreateTableAsync<HeroItem>()`) וזורעת 6 גיבורי-על התחלתיים אם הטבלה ריקה (`CountAsync() == 0`).
  4. מממש את כל 6 המתודות של `IHeroService` מול `SQLiteAsyncConnection`.

```csharp
public class SqliteHeroService : IHeroService
{
    public const string DatabaseFilename = "HeroesArchive.db3";

    public const SQLiteOpenFlags Flags =
        SQLiteOpenFlags.ReadWrite |
        SQLiteOpenFlags.Create |
        SQLiteOpenFlags.SharedCache;

    public static string DatabasePath =>
        Path.Combine(FileSystem.AppDataDirectory, DatabaseFilename);

    private static SQLiteAsyncConnection? _database;
    private static readonly SemaphoreSlim _initLock = new(1, 1);

    private async Task InitAsync()
    {
        if (_database is not null)
            return;

        await _initLock.WaitAsync();
        try
        {
            if (_database is not null)
                return;

            _database = new SQLiteAsyncConnection(DatabasePath, Flags);
            await _database.CreateTableAsync<HeroItem>();

            int existingCount = await _database.Table<HeroItem>().CountAsync();
            if (existingCount == 0)
            {
                await _database.InsertAllAsync(GetSeedHeroes());
            }
        }
        finally
        {
            _initLock.Release();
        }
    }

    public async Task<List<HeroItem>> GetAllHeroesAsync()
    {
        await InitAsync();
        return await _database!.Table<HeroItem>()
            .OrderByDescending(h => h.PowerLevel)
            .ToListAsync();
    }

    public async Task<HeroItem?> GetHeroByIdAsync(int id)
    {
        await InitAsync();
        return await _database!.Table<HeroItem>()
            .Where(h => h.Id == id)
            .FirstOrDefaultAsync();
    }

    public async Task<int> AddHeroAsync(HeroItem hero)
    {
        await InitAsync();
        return await _database!.InsertAsync(hero);
    }

    public async Task<int> UpdateHeroAsync(HeroItem hero)
    {
        await InitAsync();
        return await _database!.UpdateAsync(hero);
    }

    public async Task<int> DeleteHeroAsync(int id)
    {
        await InitAsync();
        return await _database!.DeleteAsync<HeroItem>(id);
    }
}
```

* **נקודת בדיקה לתלמיד (Checkpoint):** מדוע אנו יוצרים את הטבלה בתוך `InitAsync()` הא-סינכרונית ולא בתוך הבנאי `public SqliteHeroService()`?

---

### בלוק 4: החלפת `MockHeroService` ב-`SqliteHeroService` הודות לממשק `IHeroService`

![בלוק 4 - החלפת השירות ללא שינוי ב-ViewModels](../Images/SqliteDatabase-block4-service-swap.png)

* **קבצים ושכבה:**
  * `LearnSqlite/Views/HeroesListPage.xaml.cs`
  * `LearnSqlite/Views/AddHeroPage.xaml.cs`
  * `LearnSqlite/Views/HeroDetailsPage.xaml.cs`
* **מה הקוד עושה:** מאחר שכל שלושת ה-ViewModels (`HeroesListViewModel`, `AddHeroViewModel`, `HeroDetailsViewModel`) מקבלים בבנאי שלהם את הממשק `IHeroService` ולא מחלקה קונקרטית, **אין צורך לשנות אף שורת קוד ב-ViewModels או בקובצי ה-XAML!** כל מה שנדרש הוא להחליף את יצירת המופע מ-`new MockHeroService()` ל-`new SqliteHeroService()`:

```csharp
// ב-HeroesListPage.xaml.cs:
_viewModel = new HeroesListViewModel(new SqliteHeroService());

// ב-AddHeroPage.xaml.cs:
BindingContext = new AddHeroViewModel(new SqliteHeroService());

// ב-HeroDetailsPage.xaml.cs:
BindingContext = new HeroDetailsViewModel(new SqliteHeroService());
```

* **נקודת בדיקה לתלמיד (Checkpoint):** כיצד עבודה מול הממשק `IHeroService` הקלה עלינו במעבר מנתוני דמה בזיכרון (`MockHeroService`) למסד נתונים אמיתי (`SqliteHeroService`), ואיזה שלב עתידי נוכל להוסיף ב-`MauiProgram.cs` (Dependency Injection) כדי שאפילו קובצי ה-Code-Behind לא יצטרכו להכיר את `SqliteHeroService`?

