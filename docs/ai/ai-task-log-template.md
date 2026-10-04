# Historial de Tareas IA — Plantilla

Este documento define el formato estándar para registrar las tareas realizadas con agentes de IA en un repositorio.

**Ubicación**: Crear un archivo `docs/ai/ai-task-log.md` en cada repositorio que use agentes IA. No comprometer este archivo en este repositorio central; es un archivo de instancia por proyecto.

---

## Propósito

- Mantener trazabilidad de qué cambios fueron generados o asistidos por IA.
- Facilitar auditorías y revisiones organizacionales.
- Identificar patrones de uso, errores recurrentes y oportunidades de mejora.

---

## Formato de entrada

Cada tarea ocupa una sección con el siguiente formato:

```markdown
## YYYY-MM-DD — <Título breve de la tarea>

| Campo                    | Valor                                                  |
| ------------------------ | ------------------------------------------------------ |
| **Fecha**                | YYYY-MM-DD                                             |
| **Agente / Herramienta** | GitHub Copilot / Claude 3.5 Sonnet / ChatGPT-4o / etc. |
| **Solicitante**          | @username                                              |
| **Revisor**              | @username                                              |
| **PR / Commit**          | #123 / `abc1234`                                       |
| **Estado**               | Completado / Rechazado / Parcial                       |

## # Objetivo

Descripción en una o dos oraciones de lo que se solicitó.

## # Archivos impactados

- `src/Orders/OrderService.cs` — modificado
- `tests/Orders/OrderServiceTests.cs` — creado
- `docs/api/orders.md` — creado

## # Validaciones ejecutadas

- [x] `dotnet format --verify-no-changes` — ✅ sin errores
- [x] `dotnet build` — ✅ sin errores
- [x] `dotnet test` — ✅ 47 tests pasaron
- [x] Revisión humana — ✅ aprobado por @reviewer

## # Resultado

Descripción breve del resultado: qué se logró, qué quedó pendiente, qué problemas se encontraron.

## # Observaciones

Notas relevantes para futuras tareas: limitaciones del agente detectadas, ajustes al prompt, deuda técnica generada.
```

---

## Ejemplo completo

```markdown
## 2026-03-15 — Agregar health check endpoints

| Campo                    | Valor                            |
| ------------------------ | -------------------------------- |
| **Fecha**                | 2026-03-15                       |
| **Agente / Herramienta** | GitHub Copilot (claude-sonnet-4) |
| **Solicitante**          | @juan.perez                      |
| **Revisor**              | @maria.garcia                    |
| **PR / Commit**          | #87                              |
| **Estado**               | Completado                       |

## # Objetivo

Agregar endpoints `/health/live` y `/health/ready` en la API de pagos siguiendo
los estándares de observabilidad de microservicios de la organización.

## # Archivos impactados

- `src/Payments.Api/Program.cs` — modificado (registro de health checks)
- `src/Payments.Api/HealthChecks/DatabaseHealthCheck.cs` — creado
- `tests/Payments.Api.Tests/HealthChecksTests.cs` — creado

## # Validaciones ejecutadas

- [x] `dotnet format --verify-no-changes` — ✅ sin errores
- [x] `dotnet build --configuration Release` — ✅ sin errores
- [x] `dotnet test` — ✅ 52 tests pasaron (5 nuevos)
- [x] Revisión humana — ✅ aprobado por @maria.garcia

## # Resultado

Los endpoints funcionan correctamente. El readiness check valida la conexión
a PostgreSQL. Se cubrió el caso de BD no disponible con un test de integración
usando TestContainers.

## # Observaciones

El agente inicialmente generó el health check en la capa de Infrastructure
en lugar de la capa de Api. Se corrigió con una segunda instrucción indicando
la Clean Architecture del proyecto. Considerar agregar esta restricción al
copilot-instructions.md del repositorio.
```

---

## Instrucciones para mantener el log

1. Crear el archivo `docs/ai/ai-task-log.md` en el repositorio si no existe.
2. Agregar una entrada por cada tarea significativa completada con asistencia de IA.
3. Completar todos los campos de la tabla antes del merge del PR.
4. Las entradas se ordenan cronológicamente (más reciente primero).
5. No es necesario registrar sugerencias de autocompletado menores; solo tareas con PR o commit sustancial.
