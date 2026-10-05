# CI/CD — Guía General

Este documento define el **pipeline mínimo obligatorio** para cada tipo de proyecto, y los principios de diseño de CI/CD en la organización.

Los workflows reutilizables se centralizan en este repositorio (`.github`) y se consumen desde todos los proyectos.

---

## Principios de CI/CD

1. **CI desde el primer commit**: configurar el pipeline antes de agregar lógica.
2. **Fail fast**: los checks más rápidos primero (formato → build → tests → security).
3. **Reproducibilidad**: el mismo código debe producir el mismo resultado en cualquier ambiente.
4. **Artefactos inmutables**: el artefacto de build no se modifica entre etapas.
5. **Protección de ramas**: CI es obligatorio para merge a `main` y `develop`.

---

## Etapas del pipeline

```text
PR abierto                     Merge a develop/main       Tag v*.*.*
    │                                │                         │
    ▼                                ▼                         ▼
[Conventional Commits]        [Build + Test]           [Package + Publish]
[Code Format]                 [Security Scan]          [Deploy a Prod]
[Build + Test]                [Deploy a Integración]
```

---

## Pipeline mínimo por tipo de proyecto

## # API / Microservicio

| Etapa                | Cuándo          | Descripción                                                |
| -------------------- | --------------- | ---------------------------------------------------------- |
| Conventional Commits | PR              | Lint de todos los commits del PR                           |
| Code Format          | PR              | Verificar formato (Prettier, dotnet format, RuboCop)       |
| Build                | PR + push       | Compilar/cargar sin errores                                |
| Unit Tests           | PR + push       | Tests unitarios con coverage report                        |
| Integration Tests    | PR + push       | Tests de integración (opcional en PR, obligatorio en push) |
| Security Scan        | Push            | Escaneo de dependencias (Dependabot, bundler-audit)        |
| Package              | Tag             | Construir imagen Docker o artefacto publicable             |
| Publish              | Tag             | Publicar en registry (GHCR, Docker Hub, NuGet)             |
| Deploy               | Tag / push main | Deploy al ambiente correspondiente                         |

## # Librería / NuGet Package / Gem

| Etapa                | Cuándo    | Descripción                                   |
| -------------------- | --------- | --------------------------------------------- |
| Conventional Commits | PR        | Lint de commits                               |
| Code Format          | PR        | Verificar formato                             |
| Build                | PR + push | Compilar sin errores                          |
| Tests                | PR + push | Tests con coverage                            |
| Determine Version    | Push main | Calcular versión semántica (GitVersion)       |
| Pack + Publish       | Push main | Empaquetar y publicar en registro de paquetes |

## # Worker / Background Job

| Etapa                | Cuándo    | Descripción              |
| -------------------- | --------- | ------------------------ |
| Conventional Commits | PR        | Lint de commits          |
| Code Format          | PR        | Verificar formato        |
| Build + Test         | PR + push | Build y tests            |
| Package              | Tag       | Imagen Docker            |
| Publish + Deploy     | Tag       | Publish en GHCR + deploy |

## # Frontend / SPA (si aplica)

| Etapa                | Cuándo    | Descripción            |
| -------------------- | --------- | ---------------------- |
| Conventional Commits | PR        | Lint de commits        |
| Code Format          | PR        | Prettier, ESLint       |
| Build                | PR + push | `npm run build`        |
| Tests                | PR + push | Unit + E2E             |
| Publish              | Tag       | Deploy a CDN / hosting |

---

## Workflows reutilizables disponibles

| Workflow                                   | Referencia                                                                  |
| ------------------------------------------ | --------------------------------------------------------------------------- |
| Conventional Commits                       | `hamekoz/.github/.github/workflows/conventional-commits.yml@main`           |
| Code Format (Prettier + ShellCheck)        | `hamekoz/.github/.github/workflows/code-format-verification.yml@main`       |
| Conventions (Commits + Format)             | `hamekoz/.github/.github/workflows/conventions.yml@main`                    |
| CI .NET (format + build + test + coverage) | `hamekoz/.github/.github/workflows/dotnet.yml@main`                         |
| CD NuGet (GitHub Packages + NuGet.org)     | `hamekoz/.github/.github/workflows/continuous-delivery-nuget.yml@main`      |
| CD Docker (GHCR + Docker Hub)              | `hamekoz/.github/.github/workflows/continuous-delivery-dockerfile.yml@main` |

---

## Cómo usar un workflow reutilizable

```yaml
# .github/workflows/ci.yml en tu repositorio
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  conventions:
    uses: hamekoz/.github/.github/workflows/conventions.yml@main

  dotnet:
    needs: conventions
    uses: hamekoz/.github/.github/workflows/dotnet.yml@main
    secrets: inherit
```

---

## Secrets necesarios

| Secret               | Descripción                  | Necesario para                              |
| -------------------- | ---------------------------- | ------------------------------------------- |
| `GITHUB_TOKEN`       | Automático de GitHub Actions | Publicar paquetes en GHCR / GitHub Packages |
| `CODECOV_TOKEN`      | Token de Codecov.io          | Reportar cobertura de código                |
| `NUGET_API_KEY`      | API key de NuGet.org         | Publicar en NuGet.org                       |
| `DOCKERHUB_USERNAME` | Usuario Docker Hub           | Publicar en Docker Hub                      |
| `DOCKERHUB_TOKEN`    | Token Docker Hub             | Publicar en Docker Hub                      |

---

## Actualizaciones automáticas de dependencias

Todos los repositorios deben tener Renovate o Dependabot configurado para mantener dependencias actualizadas automáticamente.

Configuración de Renovate en este repositorio: [`.github/renovate.json`](../../.github/renovate.json).

## Seguridad y hardening

Los workflows reutilizables aplican **mínimos privilegios** (`permissions:`), **concurrency** para cancelar ejecuciones redundantes, y **timeouts** por job para evitar ejecuciones colgadas. Esto mantiene compatibilidad hacia atrás y mejora la postura de seguridad (supply chain hardening).

Recomendaciones para consumidores:
- Usar tags versionados para workflows reutilizables en producción (ej. `@v1`) en lugar de `@main` para fijar estabilidad
- Revisar los secrets requeridos en la sección anterior según destinos de publicación
- Habilitar protección de ramas y requerir checks obligatorios en PRs
