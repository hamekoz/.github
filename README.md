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

| Workflow                             | Descripción                                    |
| ------------------------------------ | ---------------------------------------------- |
| `conventions.yml`                    | Conventional Commits + verificación de formato |
| `conventional-commits.yml`           | Lint de commits con gitlint                    |
| `code-format-verification.yml`       | ShellCheck + Prettier                          |
| `dotnet.yml`                         | CI .NET: format, build, test, coverage         |
| `continuous-delivery-nuget.yml`      | CD: publicar paquetes NuGet                    |
| `continuous-delivery-dockerfile.yml` | CD: publicar imágenes Docker                   |

## Instrucciones para Copilot

Las instrucciones para GitHub Copilot válidas para toda la organización están en [`.github/copilot-instructions.md`](./.github/copilot-instructions.md).

## Perfil de la organización

Editar [`profile/README.md`](./profile/README.md) para actualizar el [perfil público de Hamekoz](https://github.com/Hamekoz/).
Ver detalles en [GitHub Documentation](https://docs.github.com/en/organizations/collaborating-with-groups-in-organizations/customizing-your-organizations-profile#organization-profile-readmes).
