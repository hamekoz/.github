# Formato de Código — .NET (C#)

Complementa las [reglas de formato comunes](./README.md). Se aplica a todos los proyectos .NET de
la organización. Refleja las prácticas promovidas por el proyecto de referencia `CartaUniversal`.

---

## Herramienta principal: `dotnet format`

El formateo automático se realiza con `dotnet format`, que aplica las reglas de `.editorconfig` y
los analizadores Roslyn. Verificación en CI:

```sh
dotnet format --verify-no-changes
```

---

## Convenciones de naming (C#)

| Elemento                      | Convención             | Ejemplo                                |
| ----------------------------- | ---------------------- | -------------------------------------- |
| Clases, Interfaces, Enums     | `PascalCase`           | `OrderService`, `IPaymentGateway`      |
| Métodos, Propiedades          | `PascalCase`           | `GetOrderById`, `IsActive`             |
| Métodos async                 | `PascalCase` + `Async` | `GetOrderAsync`                        |
| Variables locales, parámetros | `camelCase`            | `orderId`, `paymentResult`             |
| Campos privados               | `_camelCase`           | `_repository`, `_logger`               |
| Constantes                    | `UPPER_SNAKE_CASE`     | `MAX_RETRY_COUNT`, `DEFAULT_TIME_ZONE` |
| Interfaces                    | Prefijo `I`            | `IOrderRepository`                     |
| Tipos genéricos               | `T`, `TKey`, `TValue`  | `Repository<TEntity>`                  |

Las variables se nombran por **contenido**, nunca por tipo, tecnología u origen (ver
[Clean Code](../architecture/clean-code.md)).

---

## Organización de archivos y proyectos

- Un tipo (clase/interfaz/enum) por archivo.
- El nombre del archivo coincide exactamente con el nombre del tipo: `OrderService.cs`.
- Los namespaces siguen la estructura de carpetas.
- `Endpoints/` para rutas de minimal API, `Services/` para lógica de negocio, `Data/` para
  persistencia, `Models/` para DTOs.
- Usar file-scoped namespaces (C# 10+):

```csharp
namespace MyService.Application.Orders;

public class GetOrderQuery { }
```

---

## Estilos de código preferidos

```csharp
// ✅ Usar var solo cuando el tipo es evidente
var order = new Order(orderId);
var orders = repository.GetAll();

// ❌ Evitar var cuando el tipo no es claro
var result = Process(data); // ¿qué tipo es result?

// ✅ Expression-bodied members para métodos simples
public string FullName => $"{FirstName} {LastName}";

// ✅ Null-conditional y null-coalescing
var name = user?.Name ?? "Anonymous";

// ✅ Pattern matching
if (shape is Circle { Radius: > 0 } circle)
{
    // ...
}

// ✅ Records para value objects y DTOs
public record CreateOrderRequest(Guid CustomerId, List<OrderItem> Items);
```

### C# moderno — features preferidas

- **Primary constructors** para clases con mucha DI (C# 12+). Sin backing fields redundantes.
- **Collection expressions** (`[...]`, `[.. items]`) en lugar de `new List<T> { }`.
- **Records** para DTOs y value objects; property/positional pattern.
- **Switch expressions** sobre `switch` statements.
- **`var` solo cuando el tipo es evidente**; preferir tipos explícitos en el resto.

```csharp
public sealed class StockService(IStockStore store, ILogger<StockService> logger) : IStockService
{
    private readonly IStockStore _store = store; // solo si se requiere un field genuinamente
}
```

---

## Llaves y espaciado

- Siempre usar llaves en bloques `if`, `for`, `while`, incluso para una línea.
- Llave de apertura en la misma línea del bloque (Allman style opcional; reglas en `.editorconfig`).

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

- Todos los métodos async llevan sufijo `Async`: `GetOrderAsync`.
- No usar `.Result` ni `.Wait()` en código async (deadlock).
- Propagar `CancellationToken` en toda la cadena async.

```csharp
public async Task<Order> GetOrderAsync(Guid id, CancellationToken cancellationToken = default)
{
    return await _repository.FindByIdAsync(id, cancellationToken);
}
```

---

## Inyección de dependencias

```csharp
public sealed class OrderService(
    IOrderRepository repository,
    ILogger<OrderService> logger) : IOrderService
{
    // parámetros del primary constructor usados directamente
}
```

---

## Análisis estático

Recomendado para cada proyecto (`Directory.Build.props`):

```xml
<Project>

  <PropertyGroup>
    <TargetFramework>net10.0</TargetFramework>
    <LangVersion>latest</LangVersion>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>

    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
    <AnalysisLevel>latest-Default</AnalysisLevel>
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors>
  </PropertyGroup>

</Project>
```

- `EnforceCodeStyleInBuild` ejecuta las reglas de estilo IDE durante el build;
  `dotnet format --verify-no-changes` detecta drift en CI.
- `AnalysisLevel=latest-Default` habilita los analizadores del .NET SDK
  (`Microsoft.CodeAnalysis.NetAnalyzers`).
- Proyectos de larga vida pueden agregar `SonarAnalyzer.CSharp`.
- Central Package Management (`Directory.Packages.props`) con `ManagePackageVersionsCentrally=true`
  es el estándar para versiones de paquetes.

---

## global.json

Cada repositorio .NET fija el SDK en el `global.json` de la raíz para que CI y desarrollo local
usen la misma versión. Proyectos nuevos que apuntan a .NET 10:

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

La entrada `test.runner` activa el runner Microsoft Testing Platform; ver
[Testing — .NET](../testing/dotnet.md).
