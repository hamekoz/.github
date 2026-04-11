# Conventional Commits

Todos los repositorios de la organización **requieren** que los mensajes de commit sigan la especificación [Conventional Commits v1.0](https://www.conventionalcommits.org/).

El cumplimiento se verifica automáticamente en cada Pull Request mediante el workflow [`conventional-commits.yml`](../../.github/workflows/conventional-commits.yml).

## Formato

```text
<type>(<scope>): <description>

[optional body]

[optional footer(s)]
```

### Tipos permitidos

| Tipo       | Cuándo usarlo                                                          |
| ---------- | ---------------------------------------------------------------------- |
| `feat`     | Nueva funcionalidad visible para el usuario o consumidor de la API     |
| `fix`      | Corrección de un bug                                                   |
| `docs`     | Cambios solo en documentación                                          |
| `style`    | Cambios de formato que no afectan la lógica (espacios, comas, etc.)    |
| `refactor` | Reestructuración de código que no agrega funcionalidad ni corrige bugs |
| `perf`     | Mejoras de rendimiento                                                 |
| `test`     | Agregar o corregir tests                                               |
| `build`    | Cambios en el sistema de build o dependencias externas                 |
| `ci`       | Cambios en archivos y scripts de CI/CD                                 |
| `chore`    | Tareas de mantenimiento que no encajan en los anteriores               |
| `revert`   | Revertir un commit anterior                                            |

### Scope (alcance)

El scope es **opcional** pero recomendado. Debe identificar el módulo, servicio o componente afectado.

Ejemplos: `api`, `auth`, `payments`, `worker`, `ui`, `infra`, `deps`.

### Descripción

- Usar modo imperativo, tiempo presente: "add feature", no "added feature".
- No capitalizar la primera letra.
- Sin punto al final.
- Máximo 100 caracteres en el título completo.

## Breaking Changes

Para indicar un cambio que rompe la compatibilidad con versiones anteriores, agregar `!` tras el tipo/scope y un footer `BREAKING CHANGE:`:

```text
feat(api)!: remove deprecated endpoint /v1/users

BREAKING CHANGE: The /v1/users endpoint has been removed. Use /v2/users instead.
```

Los breaking changes disparan un incremento de versión MAJOR en el versionado semántico.

## Ejemplos válidos

```text
feat(auth): add OAuth2 login support
fix(payments): prevent duplicate charge on retry
docs: update contributing guide
ci: add codecov step to dotnet workflow
refactor(orders): extract discount calculator to separate class
feat(api)!: change response format for /products

BREAKING CHANGE: products endpoint now returns an array instead of a paginated object
```

## Política de squash y rebase

- Los commits individuales en una rama de trabajo **no** necesitan seguir el formato.
- Al momento de hacer merge a `main`/`develop`, **todos los commits del PR deben seguir el formato**.
- Se recomienda usar squash merge con un mensaje convencional representativo del cambio total.
- Nunca hacer force-push a ramas protegidas (`main`, `develop`).

## Configuración automática

El lint de commits se configura a través de `.gitlint` en la raíz del repositorio:

```ini
[general]
ignore=body-is-missing,body-max-line-length
contrib=contrib-title-conventional-commits

[title-max-length]
line-length=100
```

Para correr el lint localmente antes de un push:

```sh
./gitlint.sh
```
