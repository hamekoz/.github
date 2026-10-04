# Code Format — .NET (C#)

Complements the [common format rules](./README.md). Applies to every .NET project of the
organization. Reflects the practices promoted by the `CartaUniversal` reference project.

---

## Main tool: `dotnet format`

Automatic formatting uses `dotnet format`, which applies `.editorconfig` rules and Roslyn
analyzers. Verification in CI:

```sh
dotnet format --verify-no-changes
```

---

## Naming conventions (C#)

| Element                     | Convention             | Example                                |
| --------------------------- | ---------------------- | -------------------------------------- |
| Classes, Interfaces, Enums  | `PascalCase`           | `OrderService`, `IPaymentGateway`      |
| Methods, Properties         | `PascalCase`           | `GetOrderById`, `IsActive`             |
| Async methods               | `PascalCase` + `Async` | `GetOrderAsync`                        |
| Local variables, parameters | `camelCase`            | `orderId`, `paymentResult`             |
| Private fields              | `_camelCase`           | `_repository`, `_logger`               |
| Constants                   | `UPPER_SNAKE_CASE`     | `MAX_RETRY_COUNT`, `DEFAULT_TIME_ZONE` |
| Interfaces                  | `I` prefix             | `IOrderRepository`                     |
| Generic types               | `T`, `TKey`, `TValue`  | `Repository<TEntity>`                  |

Variables are named by **content**, never by type, technology, or origin (see
[Clean Code](../architecture/clean-code.md)).

---

## File and project organization

- One type (class/interface/enum) per file.
- File name matches the type name exactly: `OrderService.cs`.
- Namespaces follow the folder structure.
- `Endpoints/` for minimal API routes, `Services/` for business logic, `Data/` for persistence,
  `Models/` for DTOs.
- Use file-scoped namespaces (C# 10+):

```csharp
namespace MyService.Application.Orders;

public class GetOrderQuery { }
```

---

## Preferred code styles

```csharp
// ✅ Use var only when the type is evident
var order = new Order(orderId);
var orders = repository.GetAll();

// ❌ Avoid var when the type is unclear
var result = Process(data); // what type is result?

// ✅ Expression-bodied members for simple methods
public string FullName => $"{FirstName} {LastName}";

// ✅ Null-conditional and null-coalescing
var name = user?.Name ?? "Anonymous";

// ✅ Pattern matching
if (shape is Circle { Radius: > 0 } circle)
{
    // ...
}

// ✅ Records for value objects and DTOs
public record CreateOrderRequest(Guid CustomerId, List<OrderItem> Items);
```

## # Modern C# — preferred features

- **Primary constructors** for DI-heavy classes (C# 12+). No redundant backing fields.
- **Collection expressions** (`[...]`, `[.. items]`) instead of `new List<T> { }`.
- **Records** for DTOs and value objects; property pattern / positional pattern.
- **Switch expressions** over `switch` statements.
- **`var` only when the type is evident**; prefer explicit types otherwise.

```csharp
public sealed class StockService(IStockStore store, ILogger<StockService> logger) : IStockService
{
    private readonly IStockStore _store = store; // only when a field is genuinely required
}
```

---

## Braces and spacing

- Always use braces in `if`, `for`, `while` blocks, even for single lines.
- Opening brace on the same line as the block (Allman style optional; `.editorconfig` rules).

```csharp
public class OrderService
{
    public Order GetOrder(Guid id)
    {
        if (id == Guid.Empty)
        {
            throw new ArgumentException("Id cannot be empty.");
        }

        return _repository.FindById(id);
    }
}
```

---

## Async/Await

- All async methods carry the `Async` suffix: `GetOrderAsync`.
- Never `.Result` or `.Wait()` on async code (deadlock).
- Propagate `CancellationToken` in the whole async chain.

```csharp
public async Task<Order> GetOrderAsync(Guid id, CancellationToken cancellationToken = default)
{
    return await _repository.FindByIdAsync(id, cancellationToken);
}
```

---

## Dependency injection

```csharp
public sealed class OrderService(
    IOrderRepository repository,
    ILogger<OrderService> logger) : IOrderService
{
    // primary constructor parameters used directly
}
```

---

## Static analysis

Recommended for every project (`Directory.Build.props`):

```xml
<Project>

  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <LangVersion>latest</LangVersion>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>

    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <AnalysisLevel>latest-Default</AnalysisLevel>
    <TreatWarningsAsErrors>false</TreatWarningsAsErrors>
  </PropertyGroup>

</Project>
```

- `EnforceCodeStyleInBuild` runs IDE style rules during build; `dotnet format --verify-no-changes`
  detects drift in CI.
- `AnalysisLevel=latest-Default` enables the .NET SDK analyzers (`Microsoft.CodeAnalysis.NetAnalyzers`).
- Additional long-lived projects may add `SonarAnalyzer.CSharp`.
- Central Package Management (`Directory.Packages.props`) with `ManagePackageVersionsCentrally=true`
  is the standard for package versions.

---

## global.json

Every .NET repository pins the SDK in the root `global.json` so CI and local development use the
same version. New projects targeting .NET 10:

```json
{
  "sdk": {
    "version": "10.0.111",
    "rollForward": "latestFeature"
  },
  "test": {
    "runner": "Microsoft.Testing.Platform"
  }
}
```

The `test.runner` entry activates the Microsoft Testing Platform runner; see
[Testing — .NET](../testing/dotnet.md).
