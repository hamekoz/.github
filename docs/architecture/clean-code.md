# Clean Code

Main reference: _Clean Code_ — Robert C. Martin.

This document defines the clean-code criteria for all organization projects, in every language.
The C#-specific examples reflect the practices promoted by the `CartaUniversal` reference project.

---

## Names

### Required — Names

- Names must reveal intent: `GetUserByIdAsync` instead of `GetU` or `Process`.
- No cryptic abbreviations: `customerAccount`, not `ca` or `custAcc`.
- No generic words without meaning: `Manager`, `Processor`, `Handler`, `Helper` alone.
- Booleans are named as statements: `isActive`, `hasPermission`, `canDelete`.
- Collections are plural: `orders`, `activeUsers`.

### Required — Variables named by content, not by type

**Goal**: code reads like prose without inspecting the assignment.

```csharp
// ❌ names describe type/technology
var returnListFsharp = TzolkinDate.GetNextList(5, tzolkinDate, DateTime.UtcNow.Date);
var strVal = "user@example.com";
var tmpList = users.Where(u => u.IsActive).ToList();

// ✅ names describe content
var kinBirthdayList = TzolkinDate.GetNextList(5, tzolkinDate, DateTime.UtcNow.Date);
var userEmail = "user@example.com";
var activeUsers = users.Where(u => u.IsActive).ToList();
```

Suffixes like `Dto`, `Temp`, `List`, `Str` only appear when they genuinely aid comprehension.
Do not encode the implementation detail (framework, technology, collection kind) into the name.

### Recommended — Names

- Class names are nouns; method names are verbs.

---

## Functions / Methods

### Required — Functions

- **Single responsibility**: one function does one thing, well.
- **Small**: preferably under 20 lines. If it needs scrolling, it probably does too much.
- **One level of abstraction per function**: do not mix high-level logic with low-level details.
- **No hidden side effects**: if a function mutates external state, it must be obvious.
- **Max 3 parameters**; more suggests a parameter object or a refactor.
- **Small methods with delegated responsibilities**: extract helpers instead of god functions.

### Recommended — Functions

- Avoid boolean parameters that change behavior; prefer two separate functions.

### Required — Explicit return types over overloaded inference

Prefer `record`/class return types over tuples when the result has domain meaning.

---

## Comments

### Required — Comments

- **Code must explain itself**; comments are a last resort.
- Do not comment out obsolete code — delete it.
- Do not comment the obvious: `i++; // increment i`.

### Allowed and valuable

- Warning comments about non-obvious consequences.
- `TODO` comments with context and owner: `// TODO(juan): remove after migrating to v2`.
- Public API documentation (XML docs in .NET).
- Explanation of complex algorithms or non-evident design decisions.

---

## Format and structure

See [per-language format rules](../code-format/README.md).

### Required — Format

- Consistency across file and project.
- Related code together; unrelated code separated.
- Short lines: max 110 characters.

---

## Naming conventions (C#)

| Element | Convention | Example |
|---|---|---|
| Classes / Methods / Properties | `PascalCase` | `OrderService`, `GetOrderAsync` |
| Private fields | `_camelCase` | `_logger`, `_repository` |
| Constants | `UPPER_SNAKE_CASE` | `MAX_STOCK_ITEMS`, `DEFAULT_TIMEZONE` |
| Async methods | `Async` suffix | `GetOrderAsync` |

### Named arguments for clear intent

When calling methods with constants, `null`, or types that need inference, use named parameters:

```csharp
// ❌ what do these values mean?
var birthUtc = ToUtc(dateWithNoon, new TimeOnly(12, 0), DefaultTimeZoneId, null, null);

// ✅ clear intent
var birthUtc = ToUtc(
    date: dateWithNoon,
    timeOfBirth: new TimeOnly(hour: 12, minute: 0),
    timeZoneId: DefaultTimeZoneId,
    latitude: null,
    longitude: null);
```

---

## Error handling

### Required — Error handling

- **Never ignore exceptions silently** (`catch { }` empty is forbidden).
- Errors must carry enough context to diagnose; log with `ILogger` using structured logging
  before falling back.
- Prefer exceptions over return error codes.
- Do not use exceptions for normal control flow.

```csharp
catch (InvalidOperationException ex)
{
    _logger.LogWarning(ex, "Stock item {ItemId} not found", itemId);
    throw; // or explicit fallback
}
```

### Recommended — Error handling

- Create domain-specific exception types.
- Handle errors at the most appropriate layer (do not catch-and-rethrow without adding context).

### Structured logging

```csharp
// ❌ string concatenation — not searchable
_logger.LogInformation("User " + userId + " created chart for " + date);

// ✅ structured parameters
_logger.LogInformation("User {UserId} created chart for date {Date}", userId, date);
```

---

## Async/Await

- All async methods carry the `Async` suffix.
- Never `.Result` or `.Wait()` (deadlock risk) — async all the way.
- Propagate `CancellationToken` through the whole async chain.

```csharp
// ✅ correct
var chart = await chartService.GetChartAsync(id);
var isValid = await ValidateChartAsync(chart, cancellationToken);
```

---

## Dependency injection

- All service dependencies are injected through constructors (primary constructors preferred).
- `ILogger<T>` is required in constructor signatures of service classes; never optional/null in
  production code.
- Never instantiate `new ConcreteService()` in business code.

```csharp
public sealed class StockService(IStockStore store, ILogger<StockService> logger) : IStockService
{
    // primary constructor parameters used directly; no redundant backing fields
}
```

---

## Anti-patterns to avoid

| Anti-pattern | Problem | Solution |
|---|---|---|
| **Magic numbers/strings** | Values without context | Extract named constants |
| **God objects** | Classes doing too much | Split responsibilities |
| **Primitive obsession** | `string`/`int` instead of types | Value objects: `record OrderId(int Value)` |
| **Long parameter lists** | Methods with 5+ parameters | Data Transfer Object (DTO) |
| **Flag parameters** | `bool includeDeleted` | Separate methods: `GetActiveAsync()`, `GetDeletedAsync()` |
| **Comments instead of code** | Opaque logic + comments | Refactor to self-documenting code |

---

## Principles

### SOLID

| Principle | Summary |
|---|---|
| **S** — Single Responsibility | One class, one reason to change |
| **O** — Open/Closed | Open for extension, closed for modification |
| **L** — Liskov Substitution | Subclasses must be replaceable by their bases |
| **I** — Interface Segregation | Small, specific interfaces; no forced implementations |
| **D** — Dependency Inversion | Depend on abstractions, not concretions |

### DRY, KISS, YAGNI

| Principle | Description |
|---|---|
| **DRY** | Each piece of knowledge has one unambiguous representation |
| **KISS** | Prefer the simplest solution that works |
| **YAGNI** | Do not add functionality until needed |

---

## Code review — acceptance criteria

- [ ] Variable/method/class names are clear and consistent with the domain; variables are named
      after their content, not their type.
- [ ] Functions are small with a single responsibility.
- [ ] No dead code, commented code, or empty `catch` blocks.
- [ ] Exceptions are handled and logged with structured logging.
- [ ] Tests cover new or modified behavior.
- [ ] No avoidable duplication.
- [ ] New code introduces no circular dependencies.