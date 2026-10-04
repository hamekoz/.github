# Instrucciones para Agentes IA — Hamekoz

Este archivo define el contexto, las restricciones y el comportamiento esperado para cualquier agente de IA (GitHub Copilot, Claude, ChatGPT, etc.) que trabaje en repositorios de la organización Hamekoz.

> Fuente de verdad única del criterio: [`docs/ai/AGENTS.md`](./AGENTS.md) (inglés). Los
> repositorios referencian esa fuente con un stub en su `AGENTS.md` en lugar de duplicarla.

---

## Contexto de la organización

Hamekoz es una organización de desarrollo de software con proyectos principalmente en:

- **.NET / C#** (stack principal): APIs, microservicios, workers, librerías NuGet.
- **Ruby on Rails** (stack secundario): aplicaciones web, APIs REST.
- **Docker / GitHub Actions**: infraestructura y CI/CD centralizado.

Todos los repositorios comparten convenciones centralizadas en `hamekoz/.github`.

---

## Convenciones que el agente debe respetar siempre

## # Commits

- Todo mensaje de commit debe seguir **Conventional Commits**: `<type>(<scope>): <description>`.
- Tipos válidos: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- Sin punto final en el título. Sin mayúscula inicial. Máximo 100 caracteres en el título.

## # Branching

- Las ramas de trabajo generadas por agentes usan los mismos prefijos semánticos que los humanos,
  según el tipo de cambio: `feature/`, `fix/`, `chore/`, `docs/`, `refactor/`, `hotfix/`.
- No existe un prefijo específico para agentes (ver [Estrategia de branching](../conventions/branching.md)).
- Nunca hacer push directo a `main` o `develop`.

## # Versionado

- Seguir Semantic Versioning: `vMAJOR.MINOR.PATCH`.
- Los breaking changes requieren incremento MAJOR.

## # Idioma

- El código (nombres de variables, métodos, clases, comentarios de código) se escribe en **inglés**.
- La documentación (README, docs/, CHANGELOG) puede estar en **español o inglés** según el contexto del repositorio.

---

## Reglas de calidad de código

## # General

- Seguir los principios de **Clean Code** y **SOLID**.
- Funciones pequeñas, con una sola responsabilidad.
- Sin código comentado; eliminar el código que ya no se usa.
- Sin secretos, tokens ni credenciales en el código fuente.
- No introducir dependencias nuevas sin justificación explícita.
- No modificar archivos no relacionados con el objetivo de la tarea.

## # .NET (C#)

- Seguir las [convenciones .NET](../code-format/dotnet.md).
- `dotnet format --verify-no-changes` debe pasar sin errores.
- Usar inyección de dependencias por constructor.
- Los métodos async llevan el sufijo `Async`.

## # Ruby on Rails

- Seguir las [convenciones Ruby on Rails](../code-format/ruby-on-rails.md).
- `bundle exec rubocop` debe pasar sin errores.
- Lógica de negocio en service objects, no en controllers ni models.
- Tests con RSpec siguiendo el patrón Arrange/Act/Assert.

---

## Arquitectura

- Respetar la separación en capas de **Clean Architecture**: Domain → Application → Infrastructure → API.
- Las dependencias de código apuntan siempre hacia adentro (Domain no depende de Infrastructure).
- Aplicar los principios de **12 Factor App** especialmente en: Config (factor 3), Processes (factor 6), Logs (factor 11).

---

## Seguridad

- Nunca comprometer secretos, API keys, passwords ni tokens al repositorio.
- Usar GitHub Secrets para toda información sensible en CI/CD.
- No introducir vulnerabilidades de seguridad conocidas (inyección SQL, XSS, SSRF, etc.).
- Validar todos los inputs que vienen del exterior de la aplicación.

---

## CI/CD

- Los cambios deben pasar por el pipeline de CI antes de merge.
- No deshabilitar ni bypassear checks de CI.
- No modificar workflows de CI/CD sin justificación explícita en el PR.

---

## Restricciones de alcance

- El agente solo modifica archivos directamente relacionados con la tarea solicitada.
- No refactoriza código no relacionado con la tarea (incluso si lo considera mejorable).
- Ante ambigüedad o riesgo alto de impacto, consultar antes de actuar.

---

## Proceso de tarea

Antes de generar código, el agente debe:

1. Leer los archivos relevantes para entender el contexto actual.
2. Verificar que no exista una solución similar ya implementada.
3. Identificar y declarar qué archivos serán modificados.
4. Ejecutar lint y tests después de los cambios.
5. Registrar la tarea en el [historial de tareas IA](./ai-task-log.md) del repositorio, usando la [plantilla de log](./ai-task-log-template.md) como formato.

---

## Plantilla de solicitud

Para solicitar trabajo a un agente, usar la [plantilla de tarea IA](./task-template.md).

---

## Referencias

- [Conventional Commits](../conventions/conventional-commits.md)
- [Semantic Versioning](../conventions/semantic-versioning.md)
- [Clean Code](../architecture/clean-code.md)
- [Clean Architecture](../architecture/clean-architecture.md)
- [12 Factor](../architecture/12-factor.md)
- [Microservicios](../architecture/microservices.md)
