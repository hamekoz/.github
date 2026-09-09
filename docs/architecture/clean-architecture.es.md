# Clean Architecture

Referencia: _Clean Architecture_ — Robert C. Martin; _Domain-Driven Design_ — Eric Evans.

Este documento define el criterio de arquitectura en capas para todos los proyectos .NET de la
organización. Refleja las prácticas promovidas por el proyecto de referencia `CartaUniversal`.

---

## Principio fundamental

> **Las dependencias de código siempre apuntan hacia adentro.** Las capas externas dependen de
> las internas; nunca al revés.

```text
┌──────────────────────────────────────────────┐
│  Infrastructure / Adapters (Frameworks, DB)  │
│  ┌──────────────────────────────────────┐    │
│  │  Interface Adapters (Controllers,    │    │
│  │  Presenters, Gateways)               │    │
│  │  ┌────────────────────────────────┐  │    │
│  │  │  Application (Use Cases)       │  │    │
│  │  │  ┌──────────────────────────┐  │    │
│  │  │  │  Domain (Entities,       │  │    │
│  │  │  │  Business Rules)         │  │    │
│  │  │  └──────────────────────────┘  │    │
│  │  └────────────────────────────────┘  │    │
│  └──────────────────────────────────────┘    │
└──────────────────────────────────────────────┘
```

---

## Capas .NET concretas: cuatro capas concéntricas

Para los proyectos .NET la organización adopta cuatro capas organizadas dentro del proyecto
Web/API (separación por carpetas) o como proyectos separados según el tamaño del repositorio:

```
┌─────────────────────────────────────────────┐
│         Delivery (Web / API)                │ ← HTTP, UI, endpoints minimal API
├─────────────────────────────────────────────┤
│      Services (Use Cases / Business Logic)  │
│  <Project>/Services/                        │ ← orquesta dominio + repositorios
├─────────────────────────────────────────────┤
│         Core (Domain Entities)              │
│  <Project>.Core o <Project>/Models          │ ← tipos puros, sin dependencias
├─────────────────────────────────────────────┤
│  Data (Repositories / Persistence)          │
│  <Project>/Data/                            │ ← EF Core / Mongo / adaptadores externos
└─────────────────────────────────────────────┘
```

La referencia `CartaUniversal` mapea esto así:

- **Core** — tipos compartidos puros: enums, records, value objects, validación de dominio. Sin
  dependencias de `System.Net.Http`, ORMs ni ASP.NET.
- **Services** — casos de uso. Orquestan Core + repositorios. Inyectan abstracciones
  (`IRepository<T>`, `ILogger<T>`). Solo async, logging estructurado, manejo de errores explícito.
- **Data** — adaptadores de persistencia. Implementan interfaces definidas en Services; nunca
  exponen tipos específicos del proveedor (`IMongoCollection<T>`, `DbSet<T>`) a capas superiores.
- **Web / API** — mecanismo de entrega. Endpoints minimal API en carpeta `Endpoints/`,
  controllers/componentes Blazor, DTOs de request/response. Sin lógica de negocio y sin acceso
  directo a repositorios.

---

## Reglas por capa

### Core

- Solo tipos puros: enums, records, value objects, métodos estáticos de validación/conversión,
  excepciones de dominio (`InvalidStockOperationException`).
- Sin dependencias de proyecto, sin paquetes de infraestructura.

### Services

- Depende de tipos de Core y de abstracciones de repositorio.
- Inyección por constructor de abstracciones; nunca `new ConcreteRepository()`.
- Métodos async con sufijo `Async`; propagar `CancellationToken` en toda la cadena.
- Manejo de errores explícito — nunca tragarse excepciones.

### Data

- Implementa interfaces de Services (`IRepository<T>`).
- Mapea entre entidades de persistencia y tipos de Core/dominio.
- Sin lógica de negocio.

### Delivery (Web / API)

- Inyecta solo servicios.
- Valida entrada; delega las reglas de negocio a Services.
- Transforma DTOs → tipos de dominio → DTOs; nunca expone entidades de persistencia.
- Los minimal APIs se agrupan en clases de extensión en `Endpoints/`:

```csharp
public static class StockEndpoints
{
    public static void MapStockEndpoints(this WebApplication app)
    {
        app.MapGet("/api/stock/items/{id}", GetItem)
            .WithName("GetItem")
            .WithOpenApi();
    }

    private static async Task<IResult> GetItem(int id, IStockService stockService)
        => await stockService.GetByIdAsync(id) is { } item
            ? Results.Ok(new { Data = item })
            : Results.NotFound(new { Message = "Item not found." });
}
```

---

## Verificación de dirección de dependencias en PR

| ✅ Permitido                     | ❌ Prohibido                     |
| -------------------------------- | -------------------------------- |
| `Services` → `Core`              | `Core` → `Services`              |
| `Data` → `Services` (interfaces) | `Services` → `Data`              |
| `Web/API` → `Services`           | `Core` → `Data`                  |
| `Tests` → cualquier capa         | cualquier capa → `Tests`         |

---

## Inyección de dependencias

- Todas las dependencias de servicios se inyectan por constructor (preferir primary constructors).
- El registro de DI ocurre solo en el composition root (`Program.cs`).
- Nunca llamar `BuildServiceProvider()` durante la configuración del builder; diferir hasta después
  de `builder.Build()` (evitar `ASP0000`).

```csharp
builder.Services.AddScoped<IStockService, StockService>();
builder.Services.AddScoped<IStockStore, StockStore>();
```

---

## CQRS (Command Query Responsibility Segregation)

Recomendado para servicios con lógica de negocio no trivial:

- **Commands** modifican estado y retornan confirmación o error.
- **Queries** leen estado, sin efectos secundarios.
- Separar modelos de lectura y escritura cuando la complejidad lo justifique.

---

## Criterios de aceptación en code review

- [ ] Los tipos de dominio no referencian librerías de infraestructura.
- [ ] Los casos de uso nunca instancian repositorios; usan interfaces.
- [ ] Los endpoints/controllers solo orquestan; sin lógica de negocio.
- [ ] Las entidades de persistencia no se exponen desde la capa de entrega.
- [ ] La lógica de negocio está cubierta por tests unitarios sin base de datos ni HTTP.