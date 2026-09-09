# Clean Architecture

Reference: _Clean Architecture_ — Robert C. Martin; _Domain-Driven Design_ — Eric Evans.

This document defines the layered architecture criterion for all .NET projects of the
organization. It reflects the practices promoted by the `CartaUniversal` reference project.

---

## Core principle

> **Code dependencies always point inward.** Outer layers depend on inner layers; never the
> other way around.

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

## Concrete .NET layering: four concentric layers

For .NET projects the organization adopts four layers organized inside the API/web project
(folder separation) or as separate projects depending on the repository size:

```
┌─────────────────────────────────────────────┐
│         Delivery (Web / API)                │ ← HTTP, UI, minimal API endpoints
├─────────────────────────────────────────────┤
│      Services (Use Cases / Business Logic)  │
│  <Project>/Services/                        │ ← orchestrates domain + repositories
├─────────────────────────────────────────────┤
│         Core (Domain Entities)              │
│  <Project>.Core or <Project>/Models         │ ← pure types, no dependencies
├─────────────────────────────────────────────┤
│  Data (Repositories / Persistence)          │
│  <Project>/Data/                            │ ← EF Core / Mongo / external adapters
└─────────────────────────────────────────────┘
```

The `CartaUniversal` reference maps this as follows:

- **Core** — shared pure types: enums, records, value objects, domain validation. No
  dependencies on `System.Net.Http`, ORMs or ASP.NET.
- **Services** — use cases. Orchestrate Core + repositories. Inject abstractions
  (`IRepository<T>`, `ILogger<T>`). Async only, structured logging, explicit error handling.
- **Data** — persistence adapters. Implement interfaces defined by Services; never expose
  provider-specific types (`IMongoCollection<T>`, `DbSet<T>`) to upper layers.
- **Web / API** — delivery mechanism. Minimal API endpoints in an `Endpoints/` folder,
  controllers/Blazor components, request/response DTOs. Contains no business logic and no direct
  repository access.

---

## Rules per layer

### Core

- Only pure types: enums, records, value objects, static validation/conversion methods,
  domain exceptions (`InvalidStockOperationException`).
- No project dependencies, no infrastructure packages.

### Services

- Depends on Core types and repository abstractions.
- Constructor injection of abstractions; never `new ConcreteRepository()`.
- Methods are async with `Async` suffix; cancel `CancellationToken` in the whole chain.
- Explicit exception handling — never swallow exceptions.

### Data

- Implements Service interfaces (`IRepository<T>`).
- Maps between persistence entities and Core/domain types.
- No business logic.

### Delivery (Web / API)

- Injects services only.
- Validates input; delegates business rules to Services.
- Transforms DTOs → domain types → DTOs; never exposes persistence entities.
- Minimal APIs are grouped in `Endpoints/` extension classes:

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

## Dependency direction check in PR

| ✅ Allowed                        | ❌ Forbidden                   |
| --------------------------------- | ------------------------------ |
| `Services` → `Core`               | `Core` → `Services`            |
| `Data` → `Services` (interfaces)  | `Services` → `Data`            |
| `Web/API` → `Services`            | `Core` → `Data`                |
| `Tests` → any layer               | any layer → `Tests`            |

---

## Dependency injection

- All service dependencies are injected through constructors (primary constructors preferred).
- DI registration happens only in the composition root (`Program.cs`).
- Never call `BuildServiceProvider()` during builder configuration; defer until after
  `builder.Build()` (avoid `ASP0000`).

```csharp
builder.Services.AddScoped<IStockService, StockService>();
builder.Services.AddScoped<IStockStore, StockStore>();
```

---

## CQRS (Command Query Responsibility Segregation)

Recommended for services with non-trivial business logic:

- **Commands** modify state and return confirmation or error.
- **Queries** read state with no side effects.
- Split read/write models when complexity justifies it.

---

## Code review acceptance criteria

- [ ] Domain types do not reference infrastructure libraries.
- [ ] Use cases never instantiate repositories; they use interfaces.
- [ ] Endpoints/controllers only orchestrate; no business logic.
- [ ] Persistence entities are not exposed from the delivery layer.
- [ ] Business logic is covered by unit tests without database or HTTP.