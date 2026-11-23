# Change specification: Remove .NET browser diagnostics

## Metadata

- **Owner:** Pending maintainer assignment
- **Status:** Draft
- **Source root:** `legacy-src/dotnet-bookcatalog/shared-legacy-app`
- **Related issue:** Not created

## Problem and outcomes

### Problem

The classic BookCatalog `/Books` index reads
`HttpContext.Request.UserAgent` in `BooksController.Index` and passes the value
to the view as `ViewBag.UserAgent`
(`legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs`).
The index view renders the value with a `YOUR BROWSER:` label
(`legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml`).
This request-specific diagnostic is unrelated to catalog behavior and exposes
browser metadata that catalog users do not need.

Catalog users are affected by the response content. Maintainers are affected
by the request-data coupling that would carry into later modernization.

### Desired outcomes

1. The `/Books` page no longer reads or exposes a request user-agent value.
2. The page no longer renders a browser-diagnostic label or value.
3. The active catalog remains title-sorted and existing CRUD actions remain
   available.

### Success measures

- Source review shows no `HttpContext.Request.UserAgent`,
  `ViewBag.UserAgent`, `YOUR BROWSER:`, or equivalent browser-diagnostic
  implementation in the allowed controller and view files.
- A request with an absent, empty, normal, distinctive, or unusually long
  user-agent value receives no browser-diagnostic label or value.
- The primary build command exits successfully, or a precise environment
  blocker is recorded without claiming an implementation pass.

## Functional requirements

| ID | Requirement | Evidence or rationale |
| --- | --- | --- |
| FR-1 | The index action shall return active books ordered by `Book.Title`. | `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs:BooksController.Index` filters by `b.IsActive` and orders by `b.Title`. |
| FR-2 | The index action shall not read or pass `HttpContext.Request.UserAgent`. | `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs:BooksController.Index` currently reads the value and assigns `ViewBag.UserAgent`. |
| FR-3 | The index view shall not render a browser-diagnostic label or value. | `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml` currently renders `YOUR BROWSER:` from `ViewBag.UserAgent`. |
| FR-4 | Existing routes, list columns, status presentation, and CRUD actions shall remain unchanged. | Preserve the existing index view behavior in `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml`. |

## Constraints and out of scope

### Constraints

- **Allowed paths:**
    - `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs`
    - `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml`
- **Primary validation command:** `msbuild BookCatalog.sln -restore -p:Configuration=Release`
- **Compatibility:** Preserve ASP.NET MVC 5, .NET Framework 4.8, EF6,
  `packages.config`, and the existing solution layout.
- **Dependencies:** Introduce no package or project dependency change.
- **Data and security:** Do not store, log, pass through view data, or render
  request user-agent data. Use synthetic request values for verification.

### Exclusions (out of scope)

- ASP.NET Core or EF Core migration.
- Upgrade of .NET Framework, EF6, MVC 5, packages, or SDK/toolchain.
- Replacement browser metadata, telemetry, logging, or diagnostics.
- Styling cleanup or unrelated `System.Web` removal.
- Database data or schema changes, route or model changes, CRUD behavior
  changes, and catalog filtering or ordering changes.
- Package, configuration, test-project, and generated-artifact changes.
- Azure deployment and live production or business-database access.

## Edge cases

| Type | Condition | Expected behavior |
| --- | --- | --- |
| Normal | Active and inactive books exist | `/Books` shows only active books in title order with existing columns and CRUD links. |
| Failure | No user-agent header is present | The page renders normally with no diagnostic and no error. |
| Boundary | The user-agent value is empty or unusually long | No value or diagnostic label is rendered. |
| Privacy | A distinctive synthetic user-agent value is supplied | The response contains neither the value nor `YOUR BROWSER:`. |
| Compatibility | Existing route, model, list columns, status, and CRUD links | Behavior and presentation remain unchanged. |

## Acceptance criteria

### AC-1: Active catalog behavior is preserved

- **Given** active and inactive books exist
- **When** a user opens `/Books`
- **Then** only active books appear in title order with existing columns and
  CRUD links
- **Verification:** Perform a reproducible IIS Express/LocalDB request smoke
  check if the documented runtime prerequisites are available. If unavailable,
  report that limitation without claiming a runtime pass.

### AC-2: Browser diagnostic is absent

- **Given** a request supplies a distinctive synthetic user-agent value
- **When** a user opens `/Books`
- **Then** the response contains neither the synthetic value nor
  `YOUR BROWSER:`
- **Verification:** Check the response and search the allowed files to confirm
  that `HttpContext.Request.UserAgent`, `ViewBag.UserAgent`, and equivalent
  browser-diagnostic output are absent.

### AC-3: Missing and empty headers are safe

- **Given** a request omits or sends an empty user-agent value
- **When** the `/Books` page renders
- **Then** the page renders normally and adds no diagnostic fallback
- **Verification:** Perform reproducible request checks for missing and empty
  values when the runtime prerequisites are available.

### AC-4: Source and diff boundaries remain narrow

- **Given** the implementation is complete
- **When** repository changes are reviewed
- **Then** no change is outside the two allowed files and no package,
  configuration, database, generated, or test-project change is present
- **Verification:** `git diff --name-only <baseline>...HEAD`

### AC-5: Validation evidence is explicit

- **Given** the documented Windows prerequisites are available
- **When** the primary validation command runs from the source root
- **Then** it exits successfully
- **Verification:** Record the command, exit status, and concise result. If
  prerequisites block the command, record the exact missing tool or target
  rather than claiming success.

## Non-functional requirements

| ID | Quality | Testable requirement | Verification |
| --- | --- | --- | --- |
| NFR-1 | Privacy | User-agent data shall not be read, stored, logged, passed through view data, or rendered. | Source search and response inspection. |
| NFR-2 | Reliability | Missing or empty user-agent values shall not create an error or fallback diagnostic. | Missing-header and empty-header request checks. |
| NFR-3 | Maintainability | The change shall touch no file outside the two allowed paths and add no dependency or configuration. | Diff review with `git diff --name-only`. |
| NFR-4 | Compatibility | Existing ASP.NET MVC 5, .NET Framework 4.8, EF6, package, and solution conventions shall remain unchanged. | Diff review and primary build command. |

## Assumptions, dependencies, and risks

| Item | Type | Validation or mitigation |
| --- | --- | --- |
| The browser diagnostic has no catalog-dependent consumer. | Assumption | Repository evidence shows the index controller and view are the only code references; human review must confirm no undocumented dependency. |
| Removing the diagnostic does not require a test project. | Decision | No automated test project or standalone test target was found in the source root. Use source review and the existing runtime smoke route when prerequisites exist. |
| Visual Studio classic ASP.NET web targets may be unavailable. | Dependency / known risk | The baseline command is not a pass unless the required `Microsoft.WebApplication.targets` is available. |
| LocalDB or IIS Express may be unavailable for runtime smoke checks. | Dependency / risk | Record the exact missing prerequisite; do not claim runtime coverage. |

## Plan-mode review checkpoint

### Evidence

- **Current behavior:** `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Controllers/BooksController.cs:BooksController.Index` reads `HttpContext.Request.UserAgent` and assigns `ViewBag.UserAgent`.
- **Rendered behavior:** `legacy-src/dotnet-bookcatalog/shared-legacy-app/src/BookCatalog.Web/Views/Books/Index.cshtml` renders `YOUR BROWSER:` with `ViewBag.UserAgent`.
- **Existing tests:** No automated test project or standalone test target was found under `legacy-src/dotnet-bookcatalog/shared-legacy-app`.
- **Baseline validation evidence:** `msbuild BookCatalog.sln -restore -p:Configuration=Release`, run from the source root, exited with code 1 because `C:\Program Files\dotnet\sdk\11.0.100\Microsoft\VisualStudio\v18.0\WebApplications\Microsoft.WebApplication.targets` was not found. Restore reported no package work; source compilation did not start.

### Unresolved questions

- [ ] Human maintainer or team owner has not been assigned.
- [ ] Human reviewer name and approval date are pending.
- [ ] Approved specification commit SHA is pending.
- [ ] Runtime IIS Express/LocalDB availability is unknown.

The specification is not ready for approval until the human-review fields are
complete and runtime prerequisites or exact blockers are recorded.

### Boundary and validation review

- [ ] Evidence citations were checked against the repository.
- [ ] Allowed paths include all anticipated changes and nothing unrelated.
- [ ] The primary validation command is safe, non-interactive, and runnable
      from the stated source root.
- [ ] Acceptance criteria cover normal, failure, boundary, compatibility, and
      privacy behavior.
- [ ] Exclusions prevent unrelated modernization or cleanup.

### Stop conditions

Stop and request human guidance if:

- repository evidence contradicts this specification;
- an unresolved question could materially change behavior or scope;
- satisfying a criterion requires a path outside the allowed paths;
- the baseline or primary validation fails for a source or environment reason
  that cannot be resolved inside the approved boundary;
- the primary validation command is missing, unsafe, destructive, or
  irreproducible;
- a missing Visual Studio web target, .NET Framework targeting tool, IIS
  Express, or LocalDB prerequisite prevents required evidence;
- a secret, credential, production resource, dependency, schema change, public
  API change, or destructive action is required but not explicitly approved;
- the change would add replacement telemetry, logging, browser metadata, or
  unrelated modernization behavior; or
- implementation or validation cannot satisfy the acceptance criteria without
  expanding the approved contract.

### Human approval

- [ ] I reviewed the evidence, unresolved questions, allowed paths, acceptance
      criteria, validation command, exclusions, risks, and stop conditions.
- **Reviewer:** Pending human review
- **Approval date:** Pending human review
- **Approved specification commit:** Pending commit and human approval

Change **Status** to `Approved` only after the checkbox is selected and the
reviewer, date, and commit fields are complete. Planning approval authorizes
issue preparation, not implementation.
