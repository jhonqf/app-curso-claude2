# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

ASP.NET Core MVC web app targeting **.NET 10** (`net10.0`), built on the `dotnet new mvc` template: a single project (`app-curso-claude.csproj`) referenced by an XML-format solution file (`app-curso-claude.slnx`). No NuGet package references beyond the `Microsoft.NET.Sdk.Web` SDK, no database, no authentication scheme, and no test project.

Beyond the template's Home/Privacy/Error pages, it adds a **Contact details** page (`/Contact`) and an **Addresses** page (`/Addresses`). Their controllers return hardcoded, deliberately fictional sample data (there is no data store), and they use view models with `required`/`init` properties. UI copy is in English.

## Commands

```bash
dotnet build                          # build
dotnet run                            # run with the "http" profile -> http://localhost:5007
dotnet run --launch-profile https     # https://localhost:7291 and http://localhost:5007
dotnet watch                          # run with hot reload
```

There is no test project and no linter/formatter configuration yet. When a test project is added, include it in `app-curso-claude.slnx` and run a single test with `dotnet test --filter "FullyQualifiedName~<TestName>"`.

## Architecture

- `Program.cs` uses top-level statements and the minimal hosting model. It registers only `AddControllersWithViews()`; new services (DbContext, auth, options, etc.) are registered here before `builder.Build()`.
- Routing is conventional only: `{controller=Home}/{action=Index}/{id?}`. No attribute routing or API controllers exist.
- Static files are served with `MapStaticAssets()` and the route is chained with `.WithStaticAssets()` (the .NET 9+ build-time static asset pipeline, not `UseStaticFiles()`). Views should keep referencing assets with `asp-append-version`/`~/` paths so fingerprinting works.
- `UseAuthorization()` is in the pipeline but no authentication is configured, so `[Authorize]` will not work until a scheme is added.
- The exception handler (`/Home/Error`) and HSTS are enabled only outside the Development environment.

### Page notifications

Every page can show one Bootstrap toast on load. The pieces live in separate files:

1. A view declares the notification in its `@{ }` block:
   `ViewData["NotificationMessage"] = "..."; ViewData["NotificationType"] = "info"; // info | success | warning | danger`
2. `Views/Shared/_Layout.cshtml` renders `<partial name="_PageNotification" />` after the footer.
3. `Views/Shared/_PageNotification.cshtml` renders nothing when there is no message. It accepts only the allowlisted type values (anything else falls back to `info`) and switches to the white close icon on `success`/`danger`.
4. `wwwroot/js/site.js` shows every `.toast` on `DOMContentLoaded` via `bootstrap.Toast`. The script relies on the Bootstrap bundle loading before it.

New pages should set both `ViewData` keys. The navbar in `_Layout.cshtml` needs a link added by hand for each new top-level page.

## Conventions

- The root namespace is `app_curso_claude` (underscores), while the project, assembly, and folder names use hyphens (`app-curso-claude`). New code uses `app_curso_claude.<Folder>` namespaces, written block-scoped as in the existing files.
- Nullable reference types and implicit usings are enabled.
- `Views/_ViewImports.cshtml` already imports `app_curso_claude` and `app_curso_claude.Models` and registers the MVC tag helpers; views do not need their own `@using` for models.
- Client libraries (Bootstrap, jQuery, jquery-validation) are vendored under `wwwroot/lib/` as shipped by the template; there is no npm/LibMan manifest. Do not edit them by hand.
- `Views/Shared/_Layout.cshtml.css` is a scoped (CSS isolation) stylesheet for the layout; global styles go in `wwwroot/css/site.css`.

## Repository tooling

- `odd/tasks/*.md` hold task specs and progress logs (objective, scope, tasks, acceptance criteria, evidence). Read the relevant one before continuing a multi-step feature, and keep its checklist and evidence up to date.
- `.claude/agents/todo-reviewer.md` is a subagent that runs in an isolated git worktree. Its PreToolUse hook, `.claude/hooks/todo-reviewer-scope.js`, blocks any path outside that worktree and any Bash command other than a single read-only git command. Agent worktrees are created under `.claude/worktrees/`.
- `.claude/skills/` and `.agents/skills/` hold vendored skills tracked in `skills-lock.json`. `.atl/skill-registry.md` is generated, so do not edit it.
