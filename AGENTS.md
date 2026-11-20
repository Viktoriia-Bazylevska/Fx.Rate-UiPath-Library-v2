# UiPath Project: FX.Rates.Library.v2

Type: Library · Target: Windows · Expressions: CSharp · Studio: 26.10.0

## What this project is

- A library with one public workflow, `GetExchangeRate.xaml`, published to the tenant library feed as `FX.Rates.Library.v2`.
- Its public contract is the **Arguments** and **Behaviour** sections of `README.md`. Other processes depend on it: never rename, add, remove or retype an argument, and never change a message text, unless I ask for a breaking change.

## Working here

- Load the `uipath-rpa` skill for anything touching a workflow file. For other `uip` areas use the matching skill.
- `.xaml`, `.cs` and `project.json` are strict-schema files Studio must be able to reopen. Leave the Studio-generated view-state blocks alone and run `uip rpa validate` after every change.
- Done means `uip rpa validate` is clean, `uip rpa build` succeeds, and the Workflow Analyzer shows 0 errors (ST-MRD-009 and ST-USG-034 may stay).
- Every activity has a descriptive display name and an annotation.
- Packages: only `UiPath.System.Activities` and `UiPath.WebAPI.Activities`. Ask before adding any other: a library passes its dependencies on to every process that installs it.
- Do not publish. The developer publishes from Studio.
- Git only when I ask: check the staged files first; never commit `Data/Output/`, `.nupkg` packages, credentials, tokens or tenant URLs; use exactly the commit message I give, with no extra lines; never force-push or change remotes.
- Do not change `README.md` or `AGENTS.md`.

## Layout

- `GetExchangeRate.xaml`: the library's only public workflow and its entry point
- `project.json`: manifest (dependencies, entry point, expression language). Ground truth.
- `test-data/sample-response.json`: a saved Frankfurter response, for reference
- `.settings/`: project settings. Commit these.
- `.local/`, `.storage/`: generated. Don't edit or commit.
