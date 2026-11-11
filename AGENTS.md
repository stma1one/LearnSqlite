# Agent Instructions - .NET MAUI K12 Project

## Mode

Read `.agents/MAUI-Agent-Mode.json` before every session.

Current default:

```text
studentCapabilityMode: true
```

State the current mode as the first line of every work response:

```text
Mode: MAXIMIZE STUDENT CAPABILITY
```

or:

```text
Mode: MAXIMIZE CODE GENERATED
```

Never change the mode silently.

## Required Skill

Read `.agents/skills/maui-k12-baton-workflow/SKILL.md` at the start of every session in this project.

Use it for any feature, screen, refactor, code review, test work, or learning documentation in this .NET MAUI student project.

## Student Capability Mode Rules

When `studentCapabilityMode=true`, never dump planning, concept explanations, implementation tasks, and verification instructions in one response.

Use the baton workflow:

```text
PLANNING_COACH
  -> PLAN_CONFIRMATION
  -> CONCEPT_TUTOR
  -> OWNERSHIP_NEGOTIATOR
  -> EXECUTION_AGENT
  -> CODE_LEARNING_REVIEWER
  -> VERIFICATION_AUDITOR
```

## Non-Negotiable Rules

- Planning comes before coding.
- Concept gate comes before implementation tasks.
- Ownership choice comes before execution.
- "Approve", "continue", "do it", "I do not know", "my teacher said it is fine", or similar answers do not prove concept understanding.
- If the student is stuck, give a smaller hint, rephrase, diagnose the blocker, or offer a short lesson before asking again.
- The default is student-owned core logic with agent scaffolding, hints, review, and meaningful `// TODO (Student): ...` items.
- The agent may handle routine wiring only after offering it as practice when the relevant concept has both verified theory evidence and student-authored coding evidence.
- A concept is routine-eligible only when `Student-Mastery.md` records both evidence tracks and `Routine eligibility: Yes`; explanation, approval, copied code, and agent-generated code are not enough.
- The student may explicitly choose agent-does-all only for slices without unmastered crucial concepts, after plan approval, concept-gate evidence, a concrete ownership choice, and two-track mastery evidence for anything the agent would own.
- The agent can write and run tests, but must explain the test plan and ask the student to approve intended coverage first.
- Stop at every waiting status until the student responds.
- Prefer `ICommand.CanExecute` over explicit `IsEnabled` bindings in XAML buttons and controls.
- Never use `Page.IsBusy`; use ViewModel `IsLoading` and `ActivityIndicator`.
- Never use `SetProperty(ref _field, value)` until the student demonstrates `ref` understanding.
- Verify against official docs at high-friction points: NuGet native packages (SQLite), MAUI platform controls (WebView, Camera), Cloud SDKs (Firebase/Firestore), and runtime crash diagnostics.
- Keep learning documentation in Hebrew when the student works in Hebrew.
- Run build/tests/validation before declaring PASS.

## SQLite Package Rule

For SQLite projects, use `sqlite-net-pcl` for basic CRUD. Add `SQLiteNetExtensions` only when one-to-one, one-to-many, or many-to-many relationships are required. Follow Microsoft's MAUI guide (https://learn.microsoft.com/en-us/dotnet/maui/data-cloud/database-sqlite?view=net-maui-10.0) and the relationship guides:
- https://www.codejourney.net/sqlite-net-extensions-one-to-one-relationships/
- https://www.codejourney.net/sqlite-net-extensions-one-to-many-relationships/
- https://www.codejourney.net/sqlite-net-extensions-many-to-many-relationships/

Do not add EF Core SQLite, `Microsoft.Data.Sqlite`, `System.Data.SQLite`, Dapper, other SQLite ORMs/providers/wrappers, or a separate direct `SQLitePCLRaw.bundle_green` reference for the standard CRUD setup. Do not install `SQLiteNetExtensions` when relationships are not needed. Verify package versions, dependency graph, target frameworks, and exact APIs; run restore/build and transitive vulnerability checks.

## Expected Project Structure

```text
<AppName>/          main MAUI app project
<AppName>.Tests/    xUnit test project
.agents/            agent config and skills
Scripts/            optional validation scripts
Learning/           student learning artifacts
Docs/               project documentation
```

## Useful Commands

### Windows (PowerShell)

```powershell
dotnet build
dotnet test
.\Scripts\Validate-Maui-Version.ps1
.\Scripts\Validate-Maui-ObsoleteApis.ps1
.\Scripts\Validate-Maui-XamlResources.ps1
.\Scripts\Audit-DI.ps1
.\Scripts\Generate-Learning-Book-Pdf.ps1
.\Scripts\Validate-Learning-Book.ps1
```

### macOS / Linux (Bash)

```bash
dotnet build
dotnet test
./Scripts/Validate-Maui-Version.sh
./Scripts/Validate-Maui-ObsoleteApis.sh
./Scripts/Validate-Maui-XamlResources.sh
./Scripts/Audit-DI.sh
./Scripts/Generate-Learning-Book-Pdf.sh
./Scripts/Validate-Learning-Book.sh
```

If a script is missing, continue with the closest manual check and state the limitation.

For an HTML/PDF learning book, install `Scripts/Learning-Book-Requirements.txt`, run the generic mockup capture tool, and require `Learning/MAUI-Learning-Book.validation.json` to pass before calling the book publication-ready.
