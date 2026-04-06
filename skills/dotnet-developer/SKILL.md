---
name: dotnet-developer
description: >
  Comprehensive .NET/C# development skill for .NET 8, 9, and 10+. Covers coding standards,
  architecture, naming, async patterns, DI, testing, security, performance, functional style,
  and version-specific features. Use this skill when writing, reviewing, or refactoring any
  C# / .NET code to ensure it meets Microsoft's official conventions and modern best practices.
applyTo: ["**/*.{cs,razor,csproj}"]
skills: ["dotnet-version-checker"]
---

# .NET/C# Development Best Practices

Your task is to ensure .NET/C# code in ${selection} meets the best practices specific to this
solution/project. Apply all rules below when writing new code, reviewing existing code, or
refactoring. Detect the target framework from the `.csproj` or infer from code patterns, then
apply version-appropriate guidance.

---

## Step 1: Check .NET Version Support

Before applying guidance, use `dotnet-version-checker` to compare the codebase's target framework
or inferred .NET version with the latest supported stable release. If the local codebase version is
older than the latest supported stable release, advise that a newer version is available and
appropriate for production. If the current version is approaching end of support, mention the
support end date and recommend planning an upgrade.

### What to check

- Detect the target framework from `.csproj`, `global.json`, or source patterns.
- Run `dotnet-version-checker` to identify:
  - latest supported stable release
  - whether the current local version is supported
  - support end-of-life dates for the current version
- Use that information to tailor guidance to the appropriate runtime version.

---

## Version Reference

Always be aware of which C# language version maps to the target framework:

| .NET Version | C# Version | Key Language Features |
|-------------|-------------|----------------------|
| .NET 8 | C# 12 | Primary constructors (class/struct), collection expressions `[ ]`, inline arrays, `nameof` in attributes, interceptors (preview), default lambda parameters |
| .NET 9 | C# 13 | `params` on any collection type, `System.Threading.Lock`, partial properties, `\e` escape sequence, `ref struct` interface implementation, `allows ref struct` anti-constraint |
| .NET 10 | C# 14 | `field` keyword (semi-auto properties), `extension` blocks (extension types), `Span<char>` in pattern matching, `OverloadResolutionPriority` attribute, `null-conditional assignment` (`?.=`) |

### Key Runtime/Library Features by Version

**.NET 8:**
- Keyed dependency injection services (`[FromKeyedServices]`, `AddKeyedSingleton`)
- `FrozenDictionary<TKey, TValue>` / `FrozenSet<T>` for read-heavy lookup collections
- `TimeProvider` abstraction for testable time-dependent code
- `IExceptionHandler` middleware for centralized exception handling in ASP.NET Core
- `SearchValues<T>` for optimized multi-value searching
- Minimal API: typed results via `TypedResults`, route groups, endpoint filters
- `ConfigureHttpJsonOptions` for global JSON serialization settings

**.NET 9:**
- `HybridCache` — unified caching abstraction (replaces `IDistributedCache` + `IMemoryCache` pattern)
- `System.Threading.Lock` — lightweight, purpose-built lock type (prefer over `object` locks)
- LINQ additions: `CountBy`, `AggregateBy`, `Index`
- `JsonSchemaExporter` for generating JSON schemas from .NET types
- `TypedResults` improvements and `InternalServerError()` result
- `Guid.CreateVersion7()` for time-ordered UUIDs
- Built-in OpenAPI document generation (`Microsoft.AspNetCore.OpenApi`)

**.NET 10:**
- `field` keyword in properties — no more manual backing fields for property logic
- `extension` blocks — define extension members (methods, properties, static members) in a type block
- `System.Text.Json` source-gen improvements and `JsonNamingPolicy.SnakeCaseLower/Upper`
- `OverloadResolutionPriority` attribute for library authors to guide overload selection
- `Span<char>` and `ReadOnlySpan<char>` in pattern matching for zero-allocation string operations
- `RightJoin` LINQ operator, `LeftJoin` for EF Core

---

## Naming Conventions

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
- Verify spelling of identifiers — misspelled names are flagged

---

## Language Usage

- Prefer `var` only when the type is **obvious from the right-hand side**; never use `var` when the type is unclear without tooltips
- Use **language keywords** over runtime types: `string` not `String`, `int` not `Int32`
- Use **string interpolation** (`$""`) for short concatenations; `StringBuilder` for loops
- Prefer **raw string literals** (`"""`) over escape sequences where applicable
- Use **collection expressions** `[ ]` for array/collection initialization (.NET 8+)
- Use `Func<>` and `Action<>` instead of custom delegate types where possible
- Use `using` declarations (not `using` blocks) for `IDisposable`
- Prefer **pattern matching** over type checks + casts
- Use **nullable reference types** (NRT) — code should be NRT-aware, not sprinkled with `!`
- Use `required` properties instead of mandatory constructor params where appropriate (.NET 7+)
- Avoid `dynamic` unless interfacing with dynamic sources (COM, JSON without a model)
- Use `record` for immutable data models; `record struct` for small value types (available since C# 10)
- Use **primary constructors** for classes and structs that mainly receive DI dependencies (.NET 8+)
- Use the `field` keyword in properties instead of manual backing fields (.NET 10 / C# 14)
- Use `extension` blocks to group extension members by target type (.NET 10 / C# 14)

---

## Documentation & Structure

- Create comprehensive XML documentation comments (`/// <summary>`) for all public classes, interfaces, methods, and properties
- Include `<param>`, `<returns>`, and `<exception>` tags in XML comments
- Follow the established namespace structure: `{Core|Console|App|Service}.{Feature}`
- Inline comments should explain *why*, not *what*
- Remove dead/commented-out code — don't leave it in

---

## Design Patterns & Architecture

- Use primary constructor syntax for dependency injection: `public class MyClass(IDependency dep)`
- Implement the Command Handler pattern with generic base classes (e.g. `CommandHandler<TOptions>`)
- Use interface segregation with clear naming conventions (prefix interfaces with `I`)
- Follow the Factory pattern for complex object creation
- Follow SOLID principles — flag God classes, direct dependency on concretions, etc.
- Prefer composition over inheritance
- Seal classes that aren't designed for inheritance (`sealed` keyword)
- Keep classes focused — a class should have a single responsibility
- Keep methods focused — flag methods doing more than one thing or exceeding ~20 lines of meaningful logic
- A method should do one thing; split when responsibilities diverge

---

## Dependency Injection & Services

- Register services via DI; never `new` up injectable services inside classes
- Prefer constructor injection; avoid `IServiceLocator` / service locator anti-pattern
- Use null checks via `ArgumentNullException.ThrowIfNull()` on injected dependencies
- Register services with appropriate lifetimes (Singleton, Scoped, Transient)
- Avoid capturing scoped services in singletons (scope lifetime mismatch)
- Use `IOptions<T>` / `IOptionsSnapshot<T>` / `IOptionsMonitor<T>` for configuration rather than raw `IConfiguration` in services
- Use **keyed services** for multiple implementations of the same interface (.NET 8+):

```csharp
// Registration
builder.Services.AddKeyedSingleton<ICache, RedisCache>("redis");
builder.Services.AddKeyedSingleton<ICache, MemoryCache>("memory");

// Injection
public class MyService([FromKeyedServices("redis")] ICache cache) { }
```

- Implement service interfaces for testability

---

## Resource Management & Localization

- Use `ResourceManager` for localized messages and error strings
- Separate `LogMessages` and `ErrorMessages` resource files
- Access resources via `_resourceManager.GetString("MessageKey")`

---

## Async/Await Patterns

- Always use `async`/`await` for I/O-bound operations — **never** `.Result` or `.Wait()` (deadlock risk)
- Return `Task` or `Task<T>` from async methods; never `async void` except for event handlers
- Use `ConfigureAwait(false)` in **library code** (not required in ASP.NET Core app code)
- Use `CancellationToken` parameters in all public async methods
- Prefer `ValueTask<T>` for hot paths that frequently complete synchronously
- Handle async exceptions properly — don't let tasks go unobserved

---

## Exception Handling & Logging

- Never catch `Exception` without a filter or re-throw
- Catch the **most specific** exception type possible
- No empty catch blocks — ever
- Use `throw;` not `throw ex;` to preserve stack trace
- Don't use exceptions for control flow — use the Result pattern for expected failures (see Functional Style below)
- Use structured logging with `Microsoft.Extensions.Logging`
- Include scoped logging with meaningful context (`BeginScope`)
- Throw specific exceptions with descriptive messages
- Use `IExceptionHandler` middleware in ASP.NET Core for centralized error handling (.NET 8+)

---

## LINQ

- Prefer method syntax for simple queries, query syntax for complex multi-clause queries
- Avoid multiple enumeration of `IEnumerable<T>` — materialize with `.ToList()` or `.ToArray()` when needed
- Don't use LINQ where a simple loop is more readable
- Avoid `.Result` on async LINQ projections
- Use `CountBy`, `AggregateBy`, and `Index` for cleaner aggregation (.NET 9+)

---

## Configuration & Settings

- Use strongly-typed configuration classes with data annotations
- Implement validation attributes (`Required`, custom validators like `NotEmptyOrWhitespace`)
- Use `IConfiguration` binding to map sections to typed classes
- Support `appsettings.json` / `appsettings.{Environment}.json` configuration files
- Use `IOptionsMonitor<T>` when configuration can change at runtime

---

## Semantic Kernel & AI Integration

- Use `Microsoft.SemanticKernel` for AI operations (SK 1.x targets .NET 8+)
- Implement proper kernel configuration and service registration
- Handle AI model settings (ChatCompletion, Embedding, etc.)
- Use structured output patterns for reliable AI responses
- Register Semantic Kernel services via DI (`builder.Services.AddKernel()`)

---

## Testing Standards

- Use **Shouldly** for assertions (project standard)
- Use **Moq** for mocking dependencies
- Follow the **AAA pattern** (Arrange, Act, Assert)
- Standard naming convention: `[MethodName]_[Scenario]_[ExpectedBehavior]` (e.g. `Add_EmptyString_ReturnsZero`)
- Use realistic test data that reflects real-world scenarios (e.g. `name: "John Doe"` not `name: "test"`)
- Single logical assertion per test — verify one behavior
- Write minimally passing tests — simplest input needed to verify the behavior
- Avoid complex logic (`if`, `while`, `for`) in test code — use parameterized tests (`[DataRow]`, `[Theory]`) for multiple scenarios
- Isolate dependencies — no databases or file systems in unit tests; use DI and mocks
- Test both success and failure scenarios
- Include null parameter validation tests
- Keep unit tests in a separate project from production code
- Use `TimeProvider` for testable time-dependent logic (.NET 8+)

---

## Performance

- Avoid allocations in hot paths — flag `new` inside tight loops
- Use `Span<T>` / `Memory<T>` / `ReadOnlySpan<T>` for buffer operations where allocations matter
- Use `Span<char>` pattern matching for zero-allocation string operations (.NET 10)
- Prefer `StringBuilder` for repeated string concatenation
- Flag unnecessary `ToList()` on already-materialized collections
- Flag missing `AsNoTracking()` on EF Core read-only queries
- Use `FrozenDictionary<TKey, TValue>` / `FrozenSet<T>` for read-heavy lookup collections (.NET 8+)
- Use `SearchValues<T>` for optimized multi-value searching (.NET 8+)
- Use `System.Threading.Lock` instead of `lock(object)` for lightweight locking (.NET 9+)

---

## Security

- Flag any string interpolation or concatenation directly into SQL (SQL injection risk)
- Flag hardcoded secrets, connection strings, or credentials
- Flag missing input validation on public API endpoints
- Flag `[AllowAnonymous]` on sensitive endpoints without justification
- Use parameterized queries for all database operations
- Follow secure coding practices for AI/ML operations
- Implement proper input validation and sanitization at system boundaries

---

## Functional Style & Railway-Oriented Programming

### Extension Methods

- Prefer extension methods over static utility classes (`StringHelper.Format(x)` → `x.Format()`)
- Chain extension methods to express pipelines: `items.Filter(...).Sort(...).ToPage(...)`
- Flag helper classes like `Utils`, `Helpers`, `Manager` that are bags of static methods — convert to extension methods on the types they operate on
- Extension methods should live in a namespace that makes them opt-in, not auto-imported
- Use `extension` blocks to group related extension members (.NET 10 / C# 14)

### Result Pattern

- Avoid returning `null` to signal failure — use a `Result<T>` or `Option<T>` type
- Don't mix return values and exceptions for control flow — exceptions are for exceptional cases only
- Use the Result pattern for any method that can fail in an expected, recoverable way:

```csharp
// ❌ Avoid — null signals failure, caller must remember to check
public User? GetUser(int id) { ... }

// ✅ Prefer — failure is explicit and typed
public Result<User> GetUser(int id) { ... }
```

- If the codebase doesn't have a Result type, use a library (`FluentResults`, `ErrorOr`,
  `Ardalis.Result`, or a hand-rolled `Result<T, TError>`)
- Replace long `if/else` validation chains with railway-style chaining:

```csharp
// ❌ Avoid
if (input == null) return Error("null");
if (input.Length < 3) return Error("too short");
var parsed = Parse(input);
if (parsed == null) return Error("invalid");

// ✅ Prefer
return ValidateNotNull(input)
    .Bind(ValidateLength)
    .Bind(Parse);
```

### Functional Transformation Patterns

- Prefer `Select` / `Where` / `Aggregate` over imperative loops that build up state
- Flag mutable local variables that exist only to accumulate a result — use LINQ or a fold
- Prefer immutable data flow: avoid methods that mutate their arguments; return new values
- Flag `void` methods that modify state as a side effect when they could return the modified value
- Use `switch` expressions over `switch` statements for value-producing transformations:

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

### Pragmatic Boundaries

Don't over-apply functional style where it reduces clarity:
- Trivial one-liner methods already clear as imperative code
- Performance-critical tight loops where LINQ overhead matters
- Code that team members unfamiliar with FP will need to maintain — note the tradeoff

---

## Minimal API Patterns (.NET 8+)

- Use `TypedResults` for strongly-typed return values in endpoint handlers
- Use **route groups** to organize related endpoints and share filters/metadata
- Use **endpoint filters** for cross-cutting concerns (validation, logging)
- Return `Results<T1, T2>` union types for endpoints with multiple response types
- Use `ConfigureHttpJsonOptions` for global JSON serialization settings

---

## Code Quality

- Ensure SOLID principles compliance
- Avoid code duplication through base classes, utilities, and extension methods
- Use meaningful names that reflect domain concepts
- Keep methods focused and cohesive
- Implement proper disposal patterns for `IDisposable` / `IAsyncDisposable` resources
- Prefer `using` declarations over `using` blocks