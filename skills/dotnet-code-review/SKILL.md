---
name: dotnet-code-review
description: >
  Performs expert-level code reviews for C# and .NET 8/9/10+ code against Microsoft's official
  coding standards. Use this skill whenever the user asks to review, audit, critique, check, or
  analyze any C# or .NET code — including pasted snippets, uploaded files, or code blocks.
  Also trigger when the user asks "does this follow best practices", "is this good C#", "review
  my controller/service/repository", or any similar phrasing about code quality. The skill
  produces a structured summary report with severity ratings for all findings. It embeds
  core Microsoft standards and uses live web search to verify modern/edge-case rules.
applyTo: ["**/*.{cs, razor}"]
compatibility: ["C# 8.0+", ".NET 9+", ".NET 10+"]
skills: ["dotnet-version-checker"]
---

# .NET Code Review Skill

## Overview

This skill performs structured code reviews of C# / .NET 8+ code against Microsoft's official
coding conventions and modern .NET best practices. It produces:

1. **Summary report** — all findings grouped by severity category with line references and actionable recommendations

---

## Step 1: Identify Context and Check .NET Version Support

Before reviewing, determine:

- **Target framework**: Is this .NET 8, 9, 10+? If unspecified, assume latest stable.
- **Project type**: Web API, minimal API, console, library, Blazor, worker service?
- **Scope**: Single file, class, method, or full project?
- **Version support**: Use `dotnet-version-checker` to compare the repo's target framework or detected version against the latest supported stable .NET release.

If the user hasn't specified these, infer from the code (e.g. presence of `WebApplication.CreateBuilder`, `IHostedService`, `.csproj` `TargetFramework`, or `global.json`) and state your assumption at the top of the review.

If the detected codebase version is older than the latest supported stable release, flag that a newer supported version is available for use. If the current version is approaching end of support, note the support end date and recommend planning an upgrade in addition to the review findings.

---

## Step 2: Fetch Latest Standards (When Needed)

The core rules below are embedded in this skill. **Use web search** in these cases:

- The code uses a language feature that may be newer than C# 12 (e.g. new syntax you're uncertain about)
- The user explicitly asks to check against the latest Microsoft docs
- You're unsure whether a pattern is deprecated in the target .NET version
- The code involves APIs that change frequently (e.g. `ILogger`, `HttpClient`, `EF Core` patterns)

**Key URLs to fetch when needed:**
- `https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions`
- `https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/identifier-names`
- `https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/`
- `https://github.com/dotnet/runtime/blob/main/docs/coding-guidelines/coding-style.md`

---

## Step 3: Apply Core Review Rules

Review the code against all applicable categories below.

### Severity Scale

| Label | Meaning |
|-------|---------|
| 🔴 **Critical** | Bug risk, security issue, or major standard violation |
| 🟠 **Major** | Clear best-practice violation, maintainability concern |
| 🟡 **Minor** | Style/convention issue, readability improvement |
| 🔵 **Info** | Suggestion or modern alternative worth considering |

---

## Core Rules Reference

### Naming Conventions

- **PascalCase**: classes, interfaces, structs, records, methods, properties, events, public fields, enums, enum members, delegates
- **camelCase**: local variables, method parameters, primary constructor params (class/struct types)
- **PascalCase**: primary constructor params on `record` types
- **`_camelCase`** (underscore prefix): private and internal instance fields
- **`s_camelCase`**: private/internal static fields
- **`t_camelCase`**: thread-static fields
- **`I` prefix**: interfaces (e.g. `IWorkerQueue`)
- **`T` prefix**: generic type parameters (e.g. `TResult`, `TEntity`)
- No Hungarian notation, no type suffixes (no `strName`, `intCount`)
- No abbreviations unless well-established (`id`, `url`, `api`, `dto`)
- Async methods must end in `Async` (e.g. `GetUserAsync`)
- Verify the words are spelled correctly, Flag as 🟡 Minor Issues.
- Verify the 

### Language Usage

- Prefer `var` only when the type is **obvious from the right-hand side**; never use `var` when the type would be unclear to a reader without tooltips
- Use **language keywords** over runtime types: `string` not `String`, `int` not `Int32`, `bool` not `Boolean`
- Use **string interpolation** (`$""`) for short concatenations; `StringBuilder` for loops
- Prefer **raw string literals** (`"""`) over escape sequences where applicable
- Use **collection expressions** `[ ]` for array/collection initialization (.NET 8+)
- Use `Func<>` and `Action<>` instead of custom delegate types where possible
- Use `using` declarations (not `using` blocks with extra indentation) for `IDisposable`
- Prefer **pattern matching** over type checks + casts
- Use **nullable reference types** (NRT) — code should be NRT-aware, not just sprinkled with `!`
- Use `required` properties instead of mandatory constructor params where appropriate (.NET 7+)
- Avoid `dynamic` unless interfacing with dynamic sources (COM, JSON without a model, etc.)

### Async/Await

- Always use `async`/`await` for I/O-bound operations — never `.Result` or `.Wait()` (deadlock risk)
- Use `ConfigureAwait(false)` in library code (not required in ASP.NET Core app code)
- Avoid `async void` except for event handlers
- Return `Task` not `void` for async methods that aren't event handlers
- Use `CancellationToken` parameters in public async methods
- Prefer `ValueTask` for hot paths that frequently complete synchronously

### Exception Handling

- Never catch `Exception` without an exception filter or re-throw
- Catch the **most specific** exception type possible
- Don't swallow exceptions silently (empty catch blocks are always wrong)
- Use `throw;` not `throw ex;` to preserve stack trace
- Don't use exceptions for control flow

### LINQ

- Prefer method syntax for simple queries, query syntax for complex multi-clause queries
- Avoid multiple enumeration of `IEnumerable<T>` — call `.ToList()` or `.ToArray()` when needed
- Don't use LINQ where a simple loop is more readable
- Avoid `.Result` on async LINQ projections

### Classes and Design

- Follow SOLID principles — flag obvious violations (God classes, direct dependency on concretions, etc.)
- A method should do one thing — flag methods that are doing too much or have multiple responsibilities
- If a method is too long, split it. Flag methods longer than ~20 lines for review, but use judgment based on complexity, not just line count.
- Prefer composition over inheritance
- Seal classes that aren't designed for inheritance (`sealed` keyword)
- Keep classes focused — flag classes doing too many things
- Use `record` for immutable data models (.NET 9/10: use `record struct` for small value types)
- Mark static helpers as `static sealed` classes

### Dependency Injection (ASP.NET Core)

- Register services via DI, never use `new` for injectable services inside classes
- Prefer constructor injection; avoid `IServiceLocator` / service locator pattern
- Avoid capturing scoped services in singletons (scope lifetime mismatch)
- Use `IOptions<T>` / `IOptionsSnapshot<T>` for configuration, not raw `IConfiguration` in services

### Performance

- Avoid allocations in hot paths — flag `new` inside tight loops
- Use `Span<T>` / `Memory<T>` for buffer operations where allocations matter
- Prefer `StringBuilder` for repeated string concatenation
- Flag `ToList()` called unnecessarily on already-materialized collections
- Flag missing `AsNoTracking()` on EF Core read-only queries

### Security

- Flag any string interpolation directly into SQL (SQL injection risk)
- Flag hardcoded secrets, connection strings, or credentials
- Flag missing input validation on public API endpoints
- Flag `[AllowAnonymous]` on sensitive endpoints without comment justification

### Documentation

- Public API members should have XML doc comments (`/// <summary>`)
- Inline comments should explain *why*, not *what*
- Dead/commented-out code should be removed, not left in

---

### Functional Style & Railway-Oriented Programming

This is a first-class concern in reviews. Actively look for imperative patterns that can be
replaced with functional alternatives. Flag them as 🔵 [Info] unless they cause real bugs,
in which case escalate severity appropriately.

#### Extension Methods

- Prefer extension methods over static utility classes with `StringHelper.Format(x)` style calls
- Chain extension methods to express pipelines: `items.Filter(...).Sort(...).ToPage(...)`
- Flag helper classes like `Utils`, `Helpers`, `Manager` that are just bags of static methods —
  these should be extension methods on the types they operate on
- Extension methods should live in a namespace that makes them opt-in, not auto-imported everywhere

#### Railway-Oriented / Result Pattern

- Avoid returning `null` to signal failure — use a `Result<T>` or `Option<T>` type instead
- Flag methods that mix return values and exceptions for control flow — exceptions should be
  for exceptional cases only, not expected failure paths (e.g. "user not found" is not exceptional)
- Suggest the Result pattern for any method that can fail in an expected, recoverable way:

```csharp
// ❌ Avoid — null signals failure, caller must remember to check
public User? GetUser(int id) { ... }

// ✅ Prefer — failure is explicit and typed
public Result<User> GetUser(int id) { ... }
```

- If the codebase doesn't have a Result type, suggest adding one or using a library
  (`FluentResults`, `ErrorOr`, `Ardalis.Result`, or a simple hand-rolled `Result<T, TError>`)
- Flag long `if/else` chains that are really just sequential validation steps — these should
  be chained railway-style:

```csharp
// ❌ Avoid
if (input == null) return Error("null");
if (input.Length < 3) return Error("too short");
var parsed = Parse(input);
if (parsed == null) return Error("invalid");

// ✅ Prefer — chain validations as a pipeline
return ValidateNotNull(input)
    .Bind(ValidateLength)
    .Bind(Parse);
```

#### Functional Transformation Patterns

- Prefer `Select` / `Where` / `Aggregate` over imperative loops that build up state
- Flag mutable local variables that exist only to accumulate a result — use LINQ or a fold instead
- Prefer immutable data flow: avoid methods that mutate their arguments; return new values instead
- Flag `void` methods that modify state as a side effect when they could return the modified value
- Prefer `record` and `record struct` for data objects that flow through pipelines — immutability
  by default makes transformations safer and easier to reason about
- Use `switch` expressions over `switch` statements for transformations that produce a value:

```csharp
// ❌ Avoid
string label;
switch (status)
{
    case Status.Active: label = "Active"; break;
    case Status.Inactive: label = "Inactive"; break;
    default: label = "Unknown"; break;
}

// ✅ Prefer
var label = status switch
{
    Status.Active => "Active",
    Status.Inactive => "Inactive",
    _ => "Unknown"
};
```

#### What NOT to over-apply

Be pragmatic. Flag these as low-value functional rewrites and skip suggesting them:

- Trivial one-liner methods that are already clear as imperative code
- Performance-critical tight loops where LINQ overhead matters
- Code that other team members unfamiliar with FP will need to maintain — note the tradeoff

### Unit Tests

- Instead of "nonsense" data, use data that reflects real-world scenarios. (e.g. Instead of: name: "test" Use: name: "John Doe")
- Use `Shouldly` nuget package for assertion
- Keep unit tests in a separate project from your production code
- Arrange, Act, Assert (AAA): Organize tests into three clear phases: setting up the object (Arrange), executing the method under test (Act), and verifying the result (Assert).
- Standard Naming Convention: Use a consistent three-part name for test methods: [MethodName]_[Scenario]_[ExpectedBehavior] (e.g., Add_EmptyString_ReturnsZero).
- Isolate Dependencies: Unit tests must be independent of infrastructure like databases or file systems. Use Dependency Injection and mocking frameworks like Moq replace real dependencies with test doubles.
- Single Logical Assertion: Each test should ideally verify only one behavior. Avoid multiple "Act" tasks in a single test method.
- Write Minimally Passing Tests: Use the simplest possible input required to verify a specific behavior to keep tests resilient to future changes.
- Avoid Complex Logic: Do not use if, while, or for loops in test code. If testing multiple data scenarios, use parameterized tests (e.g., [Theory] in xUnit)

---

## Step 4: Produce the Review Output

### Format

Output the review as a single structured report. Do **not** annotate or insert comments directly into the code.

### Summary Report

Each finding must include a line reference where applicable:
- `[Line X]` for a single-line issue
- `[Lines X–Y]` for an issue spanning multiple lines
- No line reference for file-level findings (e.g. missing using directives, wrong namespace, missing XML doc on the file, overall structural concerns) — these apply to the file as a whole

```
## Code Review Summary

**Project/File:** <name>
**Target Framework:** .NET X
**Reviewed Against:** Microsoft C# Coding Conventions (fetched: <date if live fetch was done, else "embedded ruleset">)

---

### 🔴 Critical Issues (N)
- [Line X] or [Lines X–Y] Description + recommendation  *(omit line ref for file-level findings)*

### 🟠 Major Issues (N)
- [Line X] or [Lines X–Y] Description + recommendation  *(omit line ref for file-level findings)*

### 🟡 Minor Issues (N)
- [Line X] or [Lines X–Y] Description + recommendation  *(omit line ref for file-level findings)*

### 🔵 Suggestions (N)
- [Line X] or [Lines X–Y] Description + modern alternative  *(omit line ref for file-level findings)*

### 🔀 Functional / Railway Opportunities (N)
- [Line X] or [Lines X–Y] Pattern identified + recommended refactor  *(omit line ref for file-level findings)*

---

### Overall Assessment
<2-3 sentence honest summary of code quality, biggest risk areas, and whether it's production-ready>

### Top 3 Priorities to Fix
1. ...
2. ...
3. ...
```

---

## Step 5: Offer Follow-Up

After delivering the review, offer:
- "Want me to rewrite any of these sections with the fixes applied?"
- "Want me to fetch the latest Microsoft docs for any specific area (e.g. EF Core patterns, minimal APIs)?"
- "Want a deeper dive into any specific category (e.g. performance, security)?"