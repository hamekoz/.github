# Plantilla de Solicitud de Tarea para Agentes IA

Usar esta plantilla al solicitar trabajo a GitHub Copilot, Claude u otro agente IA para asegurar solicitudes claras, completas y auditables.

---

## Plantilla

```markdown
## Objetivo

<!-- Describir en una o dos oraciones qué se quiere lograr. -->

## Contexto

<!--
- ¿En qué repositorio/servicio se trabaja?
- ¿Qué módulo, feature o área se ve afectada?
- ¿Hay restricciones técnicas o de diseño relevantes?
-->

## Tarea solicitada

<!-- Descripción detallada de lo que el agente debe hacer. Ser específico:
- Qué crear, modificar o eliminar
- Qué comportamiento debe tener el resultado
- Qué no debe cambiar
-->

## Definición de terminado (DoD)

<!-- ¿Cómo se verifica que la tarea está completa y correcta? -->

- [ ] El código compila / la aplicación arranca sin errores
- [ ] Los tests existentes pasan sin modificación
- [ ] Hay tests nuevos que cubren el comportamiento agregado
- [ ] El lint pasa sin errores (`dotnet format` / `rubocop`)
- [ ] No se introdujeron secretos ni credenciales al código
- [ ] Los commits siguen Conventional Commits
- [ ] [Agregar criterios específicos del caso]

## Archivos relevantes

<!-- Listar los archivos que probablemente sean afectados o que den contexto: -->

- `src/...`
- `spec/...`

## Restricciones

<!-- ¿Qué NO debe hacer el agente? -->

- No modificar archivos fuera del scope declarado
- [Agregar restricciones específicas]

## Referencias

<!-- Links a documentación, issues, PRs, diseños, ADRs relevantes -->
```

---

## Ejemplo completo

```markdown
## Objetivo

Agregar endpoint de health check con liveness y readiness en la API de pagos.

## Contexto

Repositorio: `hamekoz/payments-api` (.NET 8, Clean Architecture)
Módulo afectado: capa de API (controllers y configuración).
El servicio ya tiene EF Core con PostgreSQL.

## Tarea solicitada

1. Crear endpoint `GET /health/live` que responda 200 si el proceso está vivo.
2. Crear endpoint `GET /health/ready` que responda 200 si la BD está accesible, 503 si no.
3. Registrar los health checks en `Program.cs` usando `IHealthChecksBuilder`.
4. No agregar autenticación a estos endpoints.

## Definición de terminado (DoD)

- [ ] El código compila sin errores
- [ ] Los tests existentes pasan
- [ ] Hay un test de integración que valida el 200 de liveness
- [ ] Hay un test de integración que valida el 503 cuando la BD no está disponible
- [ ] `dotnet format --verify-no-changes` pasa
- [ ] Los commits siguen Conventional Commits

## Archivos relevantes

- `src/Payments.Api/Program.cs`
- `src/Payments.Api/Controllers/` (referencia de estructura)
- `tests/Payments.Api.Tests/`

## Restricciones

- No modificar lógica de negocio existente
- No agregar paquetes NuGet no justificados

## Referencias

- [Guía de microservicios — Observabilidad](../docs/architecture/microservices.md#observabilidad)
- Issue #142: Add health check endpoints
```

---

## Notas para el agente

Al recibir una solicitud con esta plantilla:

1. Confirmar que se entendió el objetivo antes de escribir código.
2. Listar los archivos que se van a modificar.
3. Ejecutar lint y tests al finalizar.
4. Registrar la tarea completada en el historial de tareas IA del repositorio.
