# Hamekoz — Instrucciones para GitHub Copilot

Este archivo define el comportamiento y las restricciones para GitHub Copilot en todos los repositorios de la organización Hamekoz.

## Contexto de la organización

Hamekoz desarrolla principalmente:

- **APIs y microservicios en .NET / C#** (stack principal)
- **Aplicaciones web en Ruby on Rails** (stack secundario)
- **Infraestructura con Docker y GitHub Actions**

Las convenciones completas están en el repositorio `hamekoz/.github` dentro del directorio `docs/`.

## Commits

- Usar siempre **Conventional Commits**: `<type>(<scope>): <description>`
- Tipos válidos: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
- Sin punto final en el título. Sin mayúscula inicial. Máximo 100 caracteres.
- Las ramas generadas por Copilot usan el prefijo `copilot/`

## Idioma del código

- Código fuente (variables, métodos, clases, comentarios): **inglés**
- Documentación (README, docs, CHANGELOG): español o inglés según contexto del repositorio

## Calidad de código

- Principios **SOLID** y **Clean Code**: funciones pequeñas, una sola responsabilidad, nombres descriptivos
- Sin código comentado ni código muerto
- Sin secretos ni credenciales en el código fuente
- Solo modificar archivos directamente relacionados con la tarea

## Arquitectura

- Respetar **Clean Architecture**: las dependencias apuntan siempre hacia el dominio
- Separación estricta: Domain → Application → Infrastructure → API/Controllers
- **12 Factor App**: configuración por variables de entorno, logs a stdout, procesos stateless

## .NET (C#)

- `dotnet format --verify-no-changes` debe pasar sin errores
- Inyección de dependencias por constructor; nunca `new ConcreteService()` en código de negocio
- Métodos async con sufijo `Async`; propagar `CancellationToken`
- Campos privados con prefijo `_`: `_repository`, `_logger`

## Ruby on Rails

- `bundle exec rubocop` debe pasar sin errores
- Lógica de negocio en **service objects**, no en controllers ni models
- Tests con **RSpec**, patrón Arrange/Act/Assert
- Naming: `snake_case` para métodos/variables, `CamelCase` para clases/módulos

## Seguridad

- Nunca comprometer secrets, API keys ni tokens
- Usar GitHub Secrets para toda información sensible en CI/CD
- Validar todos los inputs externos; evitar SQL injection, XSS, SSRF
- No introducir vulnerabilidades conocidas en dependencias

## CI/CD

- No deshabilitar ni bypassear checks de CI
- No modificar workflows de CI/CD sin justificación explícita
- Los workflows reutilizables de la organización están en `hamekoz/.github`

## Proceso

Antes de generar código:

1. Leer los archivos relevantes para entender el contexto actual
2. Verificar que no exista una implementación similar ya en el código
3. Declarar explícitamente qué archivos serán modificados
4. Ejecutar lint y tests después de los cambios
5. Documentar la tarea en `docs/ai/ai-task-log.md` del repositorio (si existe)

Ante ambigüedad o riesgo de impacto alto, consultar antes de actuar.
