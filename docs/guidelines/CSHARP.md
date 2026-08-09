# Development Guidelines

This document contains critical information about working with this codebase. Follow these guidelines precisely.

## Core Development Rules

- **Read before writing.** Inspect existing project structure, `.csproj` files, and nearby code before generating anything new. Match established style over this file's defaults.
- **State the plan first** for any change touching more than one file or project. Wait for confirmation before executing.
- **Run `dotnet build` and `dotnet test`** after multi-file edits. Report results. Never claim success without compiling.
- **Don't auto-add NuGet packages.** Propose package, version, and rationale. Wait for go-ahead.
- **Don't auto-run migrations** or any data-modifying command. Generate the migration file; let me apply it.
- **When stuck, stop and ask.** Don't invent methods on third-party libraries — verify against the actual API.
- **Periodic summaries on long tasks:** what's done, what's left, decisions made unilaterally.
- **Never delete tests to make the build pass.**

## Development Philosophy

- **Correctness over cleverness.** Simple, explicit code beats clever abstractions.
- **Idiomatic .NET first.** If the framework gives you a primitive, use it before reaching for a library.
- **Domain purity.** Business logic has zero framework dependencies. No EF Core, no ASP.NET, no Mongo driver in `Domain`.
- **Result over exception** for expected failure paths (validation, not-found, conflict). Reserve exceptions for genuinely exceptional cases.
- **Async all the way down.** No sync-over-async escape hatches.
- **Composition over inheritance.** `sealed` by default; open up only with intent.
- **Vertical slices once the project grows.** Group by feature, not by technical layer, past ~20 handlers.
- **Boring is good.** Reach for novel patterns only when the existing ones demonstrably fail.

## Coding Best Practices

- **Nullable reference types enabled** project-wide (`<Nullable>enable</Nullable>`). No `!` null-forgiving operator without a comment justifying it.
- **Guard clauses early.** `ArgumentNullException.ThrowIfNull(arg)` and `ArgumentException.ThrowIfNullOrWhiteSpace(s)` at method entry.
- **`CancellationToken` on every async public method.** Propagate it all the way down.
- **`var` for locals** when the right-hand side makes the type obvious; explicit type otherwise.
- **Records for DTOs and value objects.** Classes for entities with behaviour.
- **Pattern matching** (`is`, `switch` expressions) over chained `if`/`else` and type casts.
- **LINQ:** method syntax preferred. Don't materialise mid-pipeline without a reason. Don't enumerate twice.
- **No magic numbers or strings.** Use `const`, `enum`, or named constants.
- **One public type per file.** Filename matches type name. File-scoped namespaces.
- **Naming:** PascalCase for types/methods/properties/constants; camelCase for locals/parameters; `_camelCase` for private fields; `I` prefix for interfaces.
- **Expression-bodied members** for one-liners only. Don't cram multi-line logic.
- **Options pattern** (`IOptions<T>`) over raw `IConfiguration` reads in business code. Validate on startup.
- **Structured logging** via `ILogger<T>`. No string-interpolated log messages — use templates so Serilog can index them.

## System Architecture

```
src/
  MyApp.Api/             # ASP.NET Core entry point, controllers/minimal APIs, DI wiring
  MyApp.Application/     # Use cases, command/query handlers, validators, DTOs
  MyApp.Domain/          # Entities, value objects, domain events, interfaces
  MyApp.Infrastructure/  # EF Core DbContext, repositories, external clients, migrations
  MyApp.Contracts/       # Public request/response models shared with clients (optional)
tests/
  MyApp.UnitTests/
  MyApp.IntegrationTests/
```

- **Dependency direction:** `Api -> Application -> Domain`. `Infrastructure -> Domain`. Never reverse.
- **Repository interfaces in Domain, implementations in Infrastructure.** Never leak `IQueryable<T>` past the repository boundary.
- **Cross-cutting concerns** (auth, logging, validation, error mapping) live in `Api` middleware/filters, not sprinkled through handlers.
- **No circular project references.** If you need one, the abstraction belongs in `Domain` or a new shared kernel.

## Core Components

**Stack defaults:**

- **Language:** C# 12, .NET 8 (LTS).
- **Web/API:** ASP.NET Core Minimal APIs for small services; Controllers for larger surface areas.
- **Data (relational):** Entity Framework Core 8 + PostgreSQL (Npgsql) or SQL Server.
- **Data (document):** MongoDB via official `MongoDB.Driver`.
- **Auth:** ASP.NET Core Identity (cookie); `Microsoft.AspNetCore.Authentication.JwtBearer` (token).
- **Testing:** xUnit + FluentAssertions + Moq (or NSubstitute). Integration via `WebApplicationFactory<T>` + Testcontainers.
- **Logging:** Serilog with environment-specific sinks.
- **Desktop (when needed):** WPF (Windows-only) or Avalonia (cross-platform).

**EF Core rules:**

- Migrations are code-reviewed. Never `dotnet ef database update` against shared environments without approval.
- No lazy loading. Explicit `.Include()`. `QueryTrackingBehavior.NoTracking` for read-only queries.
- `AsSplitQuery()` when including multiple collection navigations.
- Indexes declared in `OnModelCreating` or fluent config classes — not data annotations.
- `DbContext` is scoped. Never inject into a singleton.

**MongoDB rules:**

- `IMongoClient` singleton, `IMongoDatabase` singleton, collections scoped/transient.
- POCOs with `[BsonId]`/`[BsonElement]` or a central `BsonClassMap`.
- Filter/Update/Projection builders only. No string-based queries.
- Index creation in an idempotent startup initialiser, not ad-hoc.

**ASP.NET Core specifics:**

- Minimal APIs grouped via `MapGroup`. Handler delegates stay short — delegate to handler classes.
- `ProblemDetails` (RFC 7807) for error responses via a global exception handler.
- CORS, rate limiting, output caching wired in `Program.cs`.
- Health checks at `/health/live` and `/health/ready`. Ready check validates DB and downstream deps.

**Patterns to reach for:**

- **MediatR** (or a hand-rolled dispatcher) when endpoint count exceeds ~10.
- **FluentValidation** wired into the ASP.NET pipeline.
- **Result<T>** (or `OneOf`) for expected failures.

## Pull Requests

- **One logical change per PR.** Refactors separate from features. Formatting-only commits separate from logic.
- **Title format:** `<type>(<scope>): <summary>` — e.g. `feat(auth): add refresh-token rotation`. Types: `feat`, `fix`, `refactor`, `chore`, `test`, `docs`, `perf`.
- **PR description must cover:** what changed, why, how to test, any migration or config impact. Link the JIRA ticket.
- **Migrations called out explicitly** in the description with the migration name and a one-line summary of schema impact.
- **CI must be green** before review request. Don't ping reviewers on a red build.
- **No commented-out code.** Delete it; git remembers.
- **Self-review the diff first.** Catch your own debug logs, stray `TODO`s, and accidental file changes before a human does.
- **Squash on merge** unless commits are individually meaningful and curated.
- **Breaking changes flagged in the title** with `!` (e.g. `feat(api)!: rename /users endpoint`) and explained in the description.

## .NET Tools

- **`dotnet` CLI** is the source of truth. IDE shortcuts must produce the same result.
- **`dotnet format`** before commit. Should be enforced in CI.
- **`dotnet test`** with `--collect:"XPlat Code Coverage"` for coverage runs.
- **`dotnet ef migrations add <Name>`** — review the generated file before committing. Never edit applied migrations; add a new one.
- **`dotnet user-secrets`** for local secrets. Never commit `appsettings.Development.json` with real values.
- **`dotnet outdated`** (global tool) to surface stale packages periodically.

## Code Formatting

- **`.editorconfig` at repo root is authoritative.** All formatting rules live there.
- **4-space indent** for C# files; 2-space for `.csproj`, `.json`, `.yml`.
- **CRLF on Windows-only repos, LF otherwise.** Set via `.gitattributes`.
- **Braces on new lines** (Allman style) — the .NET default. Don't fight it.
- **Usings:** `System.*` first, then alphabetical. Remove unused. Outside namespace declaration.
- **Line length:** soft limit 120 chars. Break long LINQ chains one operator per line.
- **Trailing commas** in multi-line initialisers and collection expressions.
- **Blank line between methods.** No blank line at start or end of a block.
- **`dotnet format` is the tiebreaker** for any disagreement.

## Error Resolution

**Diagnosis order:**

1. **Read the full exception** — message, type, inner exceptions, stack trace. Don't pattern-match on the first line.
2. **Reproduce locally** before changing code. If you can't reproduce, instrument first; fix second.
3. **Check recent commits** in the affected area — `git log -p` on the file. Most regressions have an obvious origin.
4. **Verify assumptions** — confirm config values, connection strings, environment variables are what you think they are.
5. **Bisect when blind** — `git bisect` for regressions with an unclear cause.

**Fix discipline:**

- **Fix the cause, not the symptom.** Wrapping a flaky call in try/catch and swallowing is not a fix.
- **Add a regression test** before merging the fix. The test must fail without the fix and pass with it.
- **Don't broaden a `catch`** to silence a new exception type without understanding why it's now being thrown.
- **Log with context** — include identifiers, not just "operation failed". Use Serilog property capture.
- **`ProblemDetails` for client-facing errors.** Don't leak stack traces or internal type names across the API boundary.

**Common C# pitfalls to check first:**

- `NullReferenceException` after enabling nullable refs → trace which boundary lost the null annotation.
- `ObjectDisposedException` → check `DbContext` lifetime; usually a captured scoped service in a singleton.
- `InvalidOperationException: A second operation was started on this context` → concurrent `DbContext` use; await properly or scope per operation.
- `TaskCanceledException` mid-request → check `CancellationToken` propagation and HTTP client timeouts.
- Silent test failures → check for swallowed `Task` (missing `await`) or `async void`.

**Prohibited patterns:**

- `async void` outside event handlers.
- `.Result`, `.Wait()`, `.GetAwaiter().GetResult()`.
- Catching `Exception` without rethrowing or logging with full context.
- `Console.WriteLine` for logging in non-CLI projects.
- String concatenation in SQL or Mongo filters — parameterise.
- Hard-coded secrets, connection strings, or keys.
- Mutating shared static state.
- Disposing `HttpClient` per request — use `IHttpClientFactory`.
- Returning `IQueryable<T>` from public service methods.
- Deleting failing tests instead of fixing them.
