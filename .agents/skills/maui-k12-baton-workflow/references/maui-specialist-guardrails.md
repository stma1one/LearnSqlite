# MAUI Specialist Guardrails

Read this reference during `EXECUTION_AGENT` or `VERIFICATION_AUDITOR` when the current slice touches architecture, XAML, Shell navigation, tests, style resources, audit, or Hebrew learning documentation.

## Approved SQLite Stack (Non-Negotiable)

For SQLite student projects, use only this approved stack:
1. `sqlite-net-pcl` for basic CRUD, following https://learn.microsoft.com/en-us/dotnet/maui/data-cloud/database-sqlite?view=net-maui-10.0
2. `SQLiteNetExtensions` as an optional addition to `sqlite-net-pcl` only when one-to-one, one-to-many, or many-to-many relationships are genuinely required. Reference guides:
   - https://www.codejourney.net/sqlite-net-extensions-one-to-one-relationships/
   - https://www.codejourney.net/sqlite-net-extensions-one-to-many-relationships/
   - https://www.codejourney.net/sqlite-net-extensions-many-to-many-relationships/

Do not use EF Core SQLite, `Microsoft.Data.Sqlite`, `System.Data.SQLite`, Dapper, or another SQLite ORM/provider/wrapper. Do not directly add `SQLitePCLRaw.bundle_green` for the standard `sqlite-net-pcl` CRUD setup; use the dependency graph supplied by the approved package. Do not add `SQLiteNetExtensions` when the model has no relationships.

Implementation and verification:
- Keep database access in a Service/Repository, never in a View or ViewModel.
- Use async APIs; do not block with `.Result` or `.Wait()`.
- Verify stable package versions, target-framework compatibility, exact APIs, and dependency graphs against NuGet and the linked docs before implementation.
- Run `dotnet restore`, `dotnet build`, and `dotnet list package --vulnerable --include-transitive`.
- For relationships, identify cardinality and verify extension attributes plus loading/saving behavior; relationship loading is not assumed to happen automatically.
- If the approved packages cannot meet a requirement, explain the limitation and ask before proposing a deviation.

## Architecture And State

- Keep the project student-readable. Avoid enterprise abstractions unless they remove real complexity.
- Use clear MVVM boundaries:
  - View: layout, controls, visual states, bindings, accessibility.
  - ViewModel: UI state, commands, validation, service coordination.
  - Service: data access, APIs, persistence, platform operations.
  - Model: data representation.
- Prefer constructor dependency injection.
- Do not instantiate services inside ViewModels.
- Do not perform network, database, or file work in constructors.
- Avoid `.Result` and `.Wait()`.
- Use `async Task` for async work; reserve `async void` for required event handlers.
- Do not use `Page.IsBusy` in new code. Use ViewModel `IsLoading` plus `ActivityIndicator` or another current loading UX.
- Start students with explicit property setters. Do not use `SetProperty(ref _field, value)` until the student demonstrates `ref`, backing fields, equality checks, and `OnPropertyChanged`.
- Prefer `ICommand.CanExecute` over explicit `IsEnabled` bindings in XAML buttons and interactive controls. Use `CanExecute` with meaningful condition and refresh it via `CanExecuteChanged` when relevant state changes.

## Targeted Documentation Verification (High-Friction Triggers)

Do not perform redundant web searches for routine C# or MVVM code. 
However, you MUST actively query official documentation (Microsoft Learn for MAUI, Firebase for Google services) when encountering any of these **High-Friction Triggers**:

1. **Native & Storage NuGet Packages**:
   - When adding or choosing packages for SQLite, local storage, or platform bindings.
   - Purpose: Ensure zero package conflicts and correct platform provider bundles.
2. **Platform & Device-Specific Features**:
   - When using `WebView`, `MediaPicker`, `Geolocation`, `Permissions`, or native hardware controls.
   - Purpose: Ensure correct MAUI-specific XAML controls and lifecycle methods are used (not Web or standard desktop idioms).
3. **Cloud & Third-Party SDK Integration**:
   - When integrating Firebase, Firestore, Authentication providers, or Azure services in a MAUI client.
   - Purpose: Verify current recommended mobile/.NET SDK architecture, initialization hooks, and credentials handling.
4. **Platform Crash / Native Diagnostics**:
   - When encountering native symbol collisions, P/Invoke failures, or platform sandbox exceptions.


## XAML UI

- For Hebrew-first apps, set `FlowDirection="RightToLeft"` at Shell or page level. Do not force RTL onto URLs, emails, code, or identifiers.
- Prefer compiled bindings with `x:DataType` when the binding context is known.
- Always type `DataTemplate` contexts.
- Do not use untyped bindings to silence warnings.
- Use the simplest suitable layout: `Grid`, `VerticalStackLayout`, `HorizontalStackLayout`, `FlexLayout`, or `CollectionView`.
- Avoid arbitrary fixed page dimensions.
- Important controls need meaningful labels, readable states, and accessible touch targets.
- Prefer ViewModel validation state plus binding, `DataTrigger`, or `VisualStateManager`.
- Every `StaticResource`, `DynamicResource`, `BasedOn`, and `x:Key` reference must resolve.
- Put shared styles in `Resources/Styles/Styles.xaml`, shared colors in `Resources/Styles/Colors.xaml`, and page-specific styles in page resources.

## Shell Navigation

- Use Shell as the primary navigation system for Shell-based student projects.
- Root hierarchy belongs in `AppShell.xaml`.
- Detail pages normally use registered routes.
- Prefer route names based on `nameof(PageType)` when practical.
- Navigate with `Shell.Current.GoToAsync(...)`; back navigation uses `".."`.
- Pass small parameters through `Dictionary<string, object>`.
- Prefer passing IDs over large objects when the destination can load data itself.
- Use `IQueryAttributable` or another current documented Shell mechanism when receiving parameters.
- Register Pages and ViewModels in DI when Shell or constructors need to resolve them.
- Do not create an `INavigationService` wrapper unless tests or project requirements justify it.

## Tests

- Use xUnit for ViewModel and Service behavior.
- Use Arrange, Act, Assert.
- Prefer descriptive names such as `SaveCommand_WhenFormInvalid_CannotExecute`.
- Async tests use `async Task`.
- Test behavior, not XAML visual trees.
- Cover initial state, success, failure, validation, loading state, error state, collection updates, command behavior, and `CanExecute` where relevant.
- Use simple manual fakes when mocking libraries are not allowed.
- Reject weak tests such as `Assert.NotNull(vm)` when they cannot fail for the behavior under test.

## Verification Audit

Before declaring PASS, verify with evidence:

```text
1. Target framework/version
2. Current MAUI compatibility
3. Build
4. Architecture boundaries
5. DI
6. Navigation
7. Bindings
8. Styles/resources
9. State/commands
10. Async/error handling
11. Tests
12. Learning documentation
```

Run available checks:

**Windows (PowerShell):**
```powershell
dotnet build
dotnet test
.\Scripts\Validate-Maui-Version.ps1
.\Scripts\Validate-Maui-ObsoleteApis.ps1
.\Scripts\Validate-Maui-XamlResources.ps1
.\Scripts\Audit-DI.ps1
```

**macOS / Linux (Bash):**
```bash
dotnet build
dotnet test
./Scripts/Validate-Maui-Version.sh
./Scripts/Validate-Maui-ObsoleteApis.sh
./Scripts/Validate-Maui-XamlResources.sh
./Scripts/Audit-DI.sh
```

Never claim PASS without evidence. If a check cannot run, state the limitation.

In Student Capability Mode, also verify:
- the plan was approved or planning decisions were documented
- concept-gate evidence exists
- ownership split was chosen before execution
- agent-owned and student-owned tasks are recorded
- `Learning/Student-Mastery.md` keeps theory evidence separate from applied coding evidence
- no agent-generated, copied, or merely explained code is recorded as applied coding evidence
- agent-owned routine work is limited to concepts recorded with both evidence tracks and `Routine eligibility: Yes`

## Learning Documentation

Maintain `Learning/MAUI-Learning-Book.md` as append-only.

When the student works in Hebrew:
- write explanations in Hebrew
- keep C#, XAML, and .NET identifiers in English

For each meaningful code change, document:
- current phase
- baton workflow stage
- planning decision or concept gate
- ownership split
- what the student did
- what the agent changed
- why the architecture was chosen
- how to verify
- Do not write generic textbook lessons. Teach the actual change.

### Step-by-Step Learning Books and UI Guides

When documenting a screen, page, UI feature, or existing-project learning journey, use the `maui-step-guide-author` methodology:
- For an existing project, map current files and behavior with the student before proposing milestones to the end goal.
- Do not dump monolithic XAML or ViewModel files. Provide small source-grounded blocks with their file/layer, rationale, evidence, and a student checkpoint.
- Create an HTML visual model at `Learning/Mockups/<ScreenName>.html` only when it teaches the screen composition; label it as a model rather than emulator evidence.
- Capture the marked overview and individual blocks with `Capture-Mockup-Blocks.py`, then embed the generated images in the matching guide sections.
- Highlight visual patterns only when the actual screen uses them: layout boundaries, gradients, borders, shadows, accessibility, bindings, loading feedback, or commands.
- Match the student's working language for explanations; preserve XAML elements, C# identifiers, and commands in English.
- Save each screen guide at `Learning/Guides/<ScreenName>-Guide.md`, index it in `Learning/MAUI-Learning-Book.md`, and require the generated book validation report to pass before publication.

## Student Review Tone

Use supportive review:

```text
What is good
What to improve
Why it matters
Smallest useful fix
```

Do not help disguise generated work. Help the student understand, simplify, and explain it.
