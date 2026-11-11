# Current Phase

- **Feature:** ארכיון גיבורי-העל — מעבר מ-`MockHeroService` למסד נתונים מקומי `SQLite` (`SqliteHeroService`)
- **Phase number:** 2
- **Phase title:** חיבור מסד נתונים מקומי SQLite בענף `sqlite`
- **Primary concept:** `sqlite-net-pcl` (גרסה 1.11.285), תכונות מיפוי `[Table]` / `[PrimaryKey, AutoIncrement]` / `[Ignore]`, אתחול א-סינכרוני עצל (`InitAsync`), והחלפת שירות שקופה. חבילת `SQLiteNetExtensions` תתווסף רק בפרויקט שדורש קשרים בין טבלאות.
- **Workflow stage:** VERIFICATION_AUDITOR
- **Baton owner:** Agent (Teacher Mode -> Ready to switch back to Student Mode)
- **Approved plan:**
  1. יצירת תיקיית `docs` עם כל המדריכים וספר הלמידה.
  2. פתיחת ענף חדש `sqlite` מתוך `master`.
  3. הוספת `sqlite-net-pcl` בלבד כהפניה ישירה ל-`LearnSqlite.csproj`; אין להוסיף הפניה ישירה ל-`SQLitePCLRaw.bundle_green`. בפרויקט הדוגמה אין קשרים ולכן אין צורך ב-`SQLiteNetExtensions`.
  4. הוספת תכונות SQLite למודל `HeroItem.cs`.
  5. מימוש `SqliteHeroService.cs` המממש את `IHeroService` כולל `InitAsync()` וזריעת נתוני פתיחה (Seeding).
  6. חיבור `SqliteHeroService` בשלושת המסכים (`HeroesListPage`, `AddHeroPage`, `HeroDetailsPage`).
  7. עדכון מדריך שלב 4 וספר הלמידה בהתאם לחבילת NuGet המאושרת.
- **Verification:**
  - `dotnet restore`
  - `dotnet build`
  - `dotnet list LearnSqlite/LearnSqlite.csproj package --vulnerable --include-transitive`
  - `Validate-Maui-Version.ps1`
  - `Validate-Maui-ObsoleteApis.ps1`
  - `Validate-Maui-XamlResources.ps1`
  - `Generate-Learning-Book-Pdf.py` + `MAUI-Learning-Book.validation.json`
- **Status:** READY_TO_VERIFY
- **Verification note:** קובצי הקוד והתיעוד נבדקו סטטית דרך GitHub. `dotnet restore`, `dotnet build` והרצה על Android עדיין לא בוצעו בסביבת העבודה הזו; אין לסמן PASS עד להרצתם.
