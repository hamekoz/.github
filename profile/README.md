# Hamekoz Projects

## Estándares y convenciones

Todos los proyectos de la organización siguen los estándares definidos en el repositorio [hamekoz/.github](https://github.com/hamekoz/.github/tree/main/docs):

| Estándar                                                                                                      | Descripción                                                  |
| ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| [Conventional Commits](https://github.com/hamekoz/.github/blob/main/docs/conventions/conventional-commits.md) | Formato obligatorio de mensajes de commit                    |
| [Semantic Versioning](https://github.com/hamekoz/.github/blob/main/docs/conventions/semantic-versioning.md)   | Política de versionado `vMAJOR.MINOR.PATCH`                  |
| [Branching](https://github.com/hamekoz/.github/blob/main/docs/conventions/branching.md)                       | Estrategia de ramas: `main`, `develop`, `feature/*`, `fix/*` |
| [Clean Architecture](https://github.com/hamekoz/.github/blob/main/docs/architecture/clean-architecture.md)    | Organización Domain → Application → Infrastructure → API     |
| [12 Factor App](https://github.com/hamekoz/.github/blob/main/docs/architecture/12-factor.md)                  | Checklist por tipo de servicio                               |
| [CI/CD](https://github.com/hamekoz/.github/blob/main/docs/ci-cd/README.md)                                    | Pipelines mínimos obligatorios por tipo de proyecto          |
| [Workflows](https://github.com/hamekoz/.github/tree/main/.github/workflows)                                   | Workflows reutilizables de GitHub Actions                    |

## Requisitos mínimos para proyectos nuevos

- Solo elegir repositorios privados cuando el código tiene valor diferenciador de negocio.
- Usar la [Organización Hamekoz](https://github.com/hamekoz/) para repositorios públicos.
- No almacenar secrets en el repositorio.
- `README.md` con descripción, arquitectura mínima y guía para contribuidores.
- CI/CD configurado desde el primer commit (build + tests + verificación de convenciones).
- Branch protection en `main` y `develop` para evitar bypass del CI.

## Uso de Inteligencia Artificial

Los proyectos que usen agentes IA deben seguir la [política de uso de IA](https://github.com/hamekoz/.github/blob/main/docs/ai/agent-instructions.md), que incluye:

- Toda tarea generada por IA requiere revisión humana antes de merge.
- Registrar las tareas en el historial `docs/ai/ai-task-log.md` del repositorio.
- Las instrucciones para GitHub Copilot están centralizadas en este repositorio.

## Version.txt en APIs

Las APIs exponen un endpoint `/version` con el siguiente formato:

```text
v1.4.2+a3f9c12@https://github.com/hamekoz/my-api
```

Ver formato completo en la [guía de Semantic Versioning](https://github.com/hamekoz/.github/blob/main/docs/conventions/semantic-versioning.md#versiontxt-en-apis).

## GitHub Packages — NuGet

Para usar paquetes NuGet de la organización:

- Generar un [GitHub personal access token](https://github.com/settings/tokens/new) con permiso `read:packages`.
- Usar la variable de entorno `Hamekoz_GITHUB_PACKAGES_TOKEN` con el token generado.
- Adaptar el [`nuget.config`](https://github.com/hamekoz/.github/blob/main/dotnet-examples/nuget.config) de ejemplo al repositorio.

## Publicación automática de artefactos en tags

El workflow template `hamekoz.yml` publica automáticamente en GitHub cuando se crea un tag semver (`v1.2.3`):

## # NuGet packages

- GitHub Packages (privado, acceso mediante token)
- NuGet.org (público)

## # Docker images

- GitHub Container Registry (privado, acceso mediante token)
- Docker Hub (público, si credenciales configuradas)
