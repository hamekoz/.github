# Clean Architecture

Referencia: _Clean Architecture_ — Robert C. Martin; _Domain-Driven Design_ — Eric Evans.

---

## Principio fundamental

> **Las dependencias de código siempre apuntan hacia adentro.** Las capas externas dependen de las internas; nunca al revés.

```text
┌──────────────────────────────────────────────┐
│  Infrastructure / Adapters (Frameworks, DB)  │
│  ┌──────────────────────────────────────┐    │
│  │  Interface Adapters (Controllers,    │    │
│  │  Presenters, Gateways)               │    │
│  │  ┌────────────────────────────────┐  │    │
│  │  │  Application (Use Cases)       │  │    │
│  │  │  ┌──────────────────────────┐  │  │    │
│  │  │  │  Domain (Entities,       │  │  │    │
│  │  │  │  Business Rules)         │  │  │    │
│  │  │  └──────────────────────────┘  │  │    │
│  │  └────────────────────────────────┘  │    │
│  └──────────────────────────────────────┘    │
└──────────────────────────────────────────────┘
```

---

## Capas y responsabilidades

### 1. Domain (núcleo)

- Entidades con identidad y reglas de negocio propias.
- Value Objects: inmutables, sin identidad, comparados por valor.
- Interfaces de repositorios y servicios de dominio.
- **Sin dependencias externas** — no referencia NuGet/gems de infraestructura.

### 2. Application (casos de uso)

- Orquesta las entidades de dominio para cumplir casos de uso específicos.
- Contiene comandos (CQRS), queries, DTOs de entrada/salida.
- Define interfaces de puertos (ports) que la infraestructura implementa.
- Solo depende de `Domain`.

### 3. Interface Adapters

- Controladores de API, presenters, mappers.
- Traduce entre el modelo externo (HTTP request/response, eventos) y los DTOs de aplicación.
- Depende de `Application`.

### 4. Infrastructure

- Implementaciones concretas: repositorios de base de datos, clientes HTTP, colas de mensajes, email.
- Configuración del framework (ASP.NET, Rails).
- Depende de `Application` (implementa sus interfaces).

---

## Organización de proyectos .NET

```text
MyService/
├── MyService.Domain/          # Entidades, value objects, interfaces de repositorio
├── MyService.Application/     # Casos de uso, DTOs, interfaces de puertos
├── MyService.Infrastructure/  # EF Core, servicios externos, implementaciones
├── MyService.Api/             # Controllers, Program.cs, DI setup
└── MyService.Tests/           # Tests unitarios y de integración
```

## Organización de módulos Ruby on Rails

En Rails, la arquitectura se organiza mediante módulos y service objects:

```text
app/
├── models/          # Entidades de dominio (ActiveRecord)
├── services/        # Application layer — casos de uso
│   └── payments/
│       ├── charge_card_service.rb
│       └── refund_service.rb
├── queries/         # Queries reutilizables
├── policies/        # Autorización (Pundit)
├── serializers/     # Interface Adapters — respuestas JSON
└── controllers/     # Interface Adapters — HTTP
```

---

## Reglas de dependencia — verificación en PR

| ✅ Permitido                     | ❌ Prohibido                     |
| -------------------------------- | -------------------------------- |
| `Application` → `Domain`         | `Domain` → `Application`         |
| `Infrastructure` → `Application` | `Application` → `Infrastructure` |
| `Api` → `Application`            | `Domain` → `Infrastructure`      |
| `Tests` → cualquier capa         | cualquier capa → `Tests`         |

---

## Inyección de dependencias

- Todas las dependencias de servicios se inyectan por constructor.
- No instanciar `new ConcreteService()` en código de negocio.
- El contenedor de DI se configura únicamente en la capa de composición (`Api` / `Infrastructure`).

---

## CQRS (Command Query Responsibility Segregation)

Recomendado para servicios con lógica de negocio no trivial:

- **Commands**: modifican estado, retornan confirmación o error.
- **Queries**: leen estado, sin efectos secundarios.
- Separar modelos de lectura y escritura cuando la complejidad lo justifique.

---

## Criterios de aceptación en code review

- [ ] Las entidades de dominio no referencian librerías de infraestructura.
- [ ] Los casos de uso no instancian repositorios directamente; usan interfaces.
- [ ] Los controladores solo orquestan; no contienen lógica de negocio.
- [ ] Los DTOs están en `Application`, no en `Domain`.
- [ ] La lógica de negocio está cubierta por tests unitarios sin necesidad de base de datos ni HTTP.
