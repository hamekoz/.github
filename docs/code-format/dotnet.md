# Formato de Código — .NET (C#)

Complementa las [reglas de formato comunes](./README.md). Se aplica a todos los proyectos .NET de la organización.

---

## Herramienta principal: `dotnet format`

El formateo automático se realiza con `dotnet format`, que aplica las reglas de `.editorconfig` y los analizadores Roslyn.

Verificación en CI:

```sh
dotnet format --verify-no-changes
```

---

## Convenciones de naming (C#)

| Elemento                      | Convención                   | Ejemplo                           |
| ----------------------------- | ---------------------------- | --------------------------------- |
| Clases, Interfaces, Enums     | `PascalCase`                 | `OrderService`, `IPaymentGateway` |
| Métodos, Propiedades          | `PascalCase`                 | `GetOrderById`, `IsActive`        |
| Variables locales, parámetros | `camelCase`                  | `orderId`, `paymentResult`        |
| Campos privados               | `_camelCase` con prefijo `_` | `_repository`, `_logger`          |
| Constantes                    | `PascalCase`                 | `MaxRetryCount`                   |
| Interfaces                    | Prefijo `I`                  | `IOrderRepository`                |
| Tipos genéricos               | `T`, `TKey`, `TValue`        | `Repository<TEntity>`             |

---

## Organización de archivos y proyectos

- Un tipo (clase/interfaz/enum) por archivo.
- El nombre del archivo coincide exactamente con el nombre del tipo: `OrderService.cs`.
- Los namespaces siguen la estructura de carpetas.
- Usar file-scoped namespaces (C# 10+):

```csharp
namespace MyService.Application.Orders;

public class GetOrderQuery { }
```

---

## Estilos de código preferidos

```csharp
// ✅ Usar var cuando el tipo es evidente
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

---

## Llaves y espaciado

- Siempre usar llaves en bloques `if`, `for`, `while`, incluso para una línea.
- Llave de apertura en la misma línea del bloque en clases y métodos (Allman style para .NET).

```csharp
// ✅ Correcto (Allman style)
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
- Propagar `CancellationToken` en toda la cadena async cuando sea posible.

```csharp
// ✅ Correcto
public async Task<Order> GetOrderAsync(Guid id, CancellationToken cancellationToken = default)
{
    return await _repository.FindByIdAsync(id, cancellationToken);
}
```

---

## Inyección de dependencias

```csharp
// ✅ Inyección por constructor (obligatorio)
public class OrderService
{
    private readonly IOrderRepository _repository;
    private readonly ILogger<OrderService> _logger;

    public OrderService(IOrderRepository repository, ILogger<OrderService> logger)
    {
        _repository = repository;
        _logger = logger;
    }
}
```

---

## Cobertura de análisis estático

Configurar los siguientes analizadores en los proyectos:

- `Microsoft.CodeAnalysis.NetAnalyzers` (incluido en .NET SDK).
- `SonarAnalyzer.CSharp` (recomendado para proyectos de larga vida).
- Advertencias tratadas como errores en CI (`<TreatWarningsAsErrors>true</TreatWarningsAsErrors>`).

---

## global.json

Cada repositorio .NET debe tener un `global.json` en la raíz para fijar la versión del SDK:

```json
{
  "sdk": {
    "version": "8.0.204",
    "rollForward": "latestMinor"
  }
}
```

Esto garantiza que CI y desarrollo local usen la misma versión del SDK.
