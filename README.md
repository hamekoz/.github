# Hamekoz — Repositorio de Estándares Organizacionales

Este repositorio centraliza la documentación, convenciones y workflows compartidos para **todos los repositorios de la organización Hamekoz**.

## Documentación centralizada

Toda la documentación de estándares y convenciones está en el directorio [`docs/`](./docs/README.md):

- **[Conventional Commits](./docs/conventions/conventional-commits.md)** — formato obligatorio de commits
- **[Semantic Versioning](./docs/conventions/semantic-versioning.md)** — política de versionado
- **[Branching](./docs/conventions/branching.md)** — estrategia de ramas y ambientes
- **[Clean Architecture](./docs/architecture/clean-architecture.md)** — organización en capas
- **[Clean Code](./docs/architecture/clean-code.md)** — criterios de código limpio
- **[12 Factor App](./docs/architecture/12-factor.md)** — checklist por tipo de servicio
- **[Microservicios](./docs/architecture/microservices.md)** — principios y límites de contexto
- **[Formato .NET](./docs/code-format/dotnet.md)** y **[Rails](./docs/code-format/ruby-on-rails.md)**
- **[CI/CD](./docs/ci-cd/README.md)** — pipelines mínimos y workflows reutilizables
- **[Instrucciones para agentes IA](./docs/ai/agent-instructions.md)** — política de uso de IA
- **[Onboarding de repositorios](./docs/adoption/onboarding-checklist.md)**

## Workflows reutilizables

Los workflows de GitHub Actions compartidos están en [`.github/workflows/`](./.github/workflows/):

## # Workflows de validación y calidad

| Workflow                       | Descripción                                    |
| ------------------------------ | ---------------------------------------------- |
| `conventions.yml`              | Conventional Commits + verificación de formato |
| `conventional-commits.yml`     | Lint de commits con gitlint                    |
| `code-format-verification.yml` | ShellCheck + Prettier                          |
| `dotnet.yml`                   | CI .NET: format, build, test, coverage         |

## # Workflows de publicación de NuGet

| Workflow                                      | Descripción                                       |
| --------------------------------------------- | ------------------------------------------------- |
| `continuous-delivery-nuget.yml` (coordinador) | Orquesta publicación en ambos destinos            |
| `publish-nuget-to-github-packages.yml`        | Publica en GitHub Packages (nuget.pkg.github.com) |
| `publish-nuget-to-nuget-org.yml`              | Publica en NuGet.org (api.nuget.org)              |

**Características**:

- Workflows independientes invocables por separado
- Input opcional `nuget-version` para especificar versión manual
- Sin input, determina versión automáticamente con GitVersion
- Retrocompatible con flujos existentes

## # Workflows de publicación de Docker

| Workflow                                           | Descripción                                    |
| -------------------------------------------------- | ---------------------------------------------- |
| `continuous-delivery-dockerfile.yml` (coordinador) | Orquesta publicación en ambos destinos         |
| `publish-docker-image-to-github-packages.yml`      | Publica en GitHub Container Registry (ghcr.io) |
| `publish-docker-image-to-dockerhub.yml`            | Publica en Docker Hub (docker.io)              |

**Características**:

- Workflows independientes invocables por separado
- Soporta múltiples proyectos con configuración de paths

## Instrucciones para Copilot

Las instrucciones para GitHub Copilot válidas para toda la organización están en [`.github/copilot-instructions.md`](./.github/copilot-instructions.md).

## Workflow template

El template de workflow [`workflow-templates/hamekoz.yml`](./workflow-templates/hamekoz.yml) automatiza completamente el CI/CD en repositorios:

- ✅ Validación de commits y formato en PRs
- ✅ CI/CD en push a ramas (main, uat, stg)
- ✅ Publicación automática de NuGet en tags semver
- ✅ Publicación automática de Docker en tags semver
- ✅ Jobs de ejemplo para múltiples proyectos

**Activación**: Crear `github/workflows/hamekoz.yml` en tu repositorio con:

```yaml
name: Hamekoz
on:
  pull_request:
    branches: [$default-branch]
  push:
    branches: [$default-branch, "uat", "stg"]
    tags:
      - v*.*.*

jobs:
  hamekoz:
    uses: hamekoz/.github/.github/workflows/hamekoz.yml@main
```

## Perfil de la organización

Editar [`profile/README.md`](./profile/README.md) para actualizar el [perfil público de Hamekoz](https://github.com/Hamekoz/).
Ver detalles en [GitHub Documentation](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/customizing-your-organizations-profile#organization-profile-readmes).

## Templates and Governance

- **CODEOWNERS**: `.github/CODEOWNERS` — ownership rules for automated review assignment
- **Issue Templates**: `.github/ISSUE_TEMPLATE/` — bug, feature, and chore templates
- **PR Template**: `.github/PULL_REQUEST_TEMPLATE.md` — enforces Conventional Commits and checklist
- **Security Policy**: `SECURITY.md` — private vulnerability reporting via GitHub Security Advisories
- **Pre-commit**: `.pre-commit-config.yaml` — local quality checks (conventional commits, format, markdown, spell)
