# CI/CD — .NET

Guía detallada de los workflows de CI/CD para proyectos .NET de la organización.

Ver la [guía general de CI/CD](./README.md) para el pipeline mínimo y los principios.

---

## Workflows disponibles

## # 1. `dotnet.yml` — Integración Continua

Verifica formato, compila, ejecuta tests y reporta cobertura.

```yaml
jobs:
  dotnet:
    uses: hamekoz/.github/.github/workflows/dotnet.yml@main
    secrets: inherit
```

**Pasos que ejecuta:**

1. Checkout con historial completo (`fetch-depth: 0`).
2. Setup .NET con la versión de `global.json`.
3. `dotnet format --verify-no-changes` — falla si el código no está formateado. Personalizable vía el input `format_command`.
4. `dotnet build --configuration Release` — falla si hay errores de compilación.
5. `dotnet test --no-build --configuration Release --collect:"XPlat Code Coverage"`.
6. Subir reporte de cobertura a Codecov.

**Inputs:**

- `format_command`: comando de verificación de formato. El default es `dotnet format --verify-no-changes`
  (todas las capas: whitespace, style y analyzers). Los repositorios con código legacy y/o soluciones
  grandes pueden limitar la capa de analizadores al código nuevo:

```yaml
jobs:
  dotnet:
    uses: hamekoz/.github/.github/workflows/dotnet.yml@main
    secrets: inherit
    with:
      format_command: >-
        dotnet format whitespace --verify-no-changes
        && dotnet format style --verify-no-changes
        && dotnet format analyzers --verify-no-changes
  - -include "src/**" --include "tests/**"
```

> Nota: el `--verify-no-changes` de la capa `analyzers` aplica, entre otros, el arreglo
> `[Obsolete]` de Roslyn sobre código legacy que usa APIs obsoletas (p.ej. en .NET 10
> `System.Data.SqlClient`). Por eso en repositorios legacy conviene scoping nuevo. Por el mismo
> motivo, el código nuevo debe usar APIs no obsoletas en vez de depender de excepciones.

## # 2. `continuous-delivery-nuget.yml` — Publicar paquetes NuGet

Versiona, empaqueta y publica en GitHub Packages y/o NuGet.org.

```yaml
jobs:
  publish:
    uses: hamekoz/.github/.github/workflows/continuous-delivery-nuget.yml@main
    secrets: inherit
```

**Pasos que ejecuta:**

1. Setup .NET con versión de `global.json`.
2. Instalar GitVersion y calcular la versión semántica automáticamente.
3. `dotnet build --configuration Release`.
4. `dotnet pack --output ./nugets -p:Version=${{ steps.version.outputs.nuGetVersion }}`.
5. Publicar en GitHub Packages (con `GITHUB_TOKEN`).
6. Publicar en NuGet.org (con `NUGET_API_KEY`).

## # 3. `continuous-delivery-dockerfile.yml` — Publicar imagen Docker

Construye y publica imagen Docker en GHCR y/o Docker Hub.

```yaml
jobs:
  docker:
    uses: hamekoz/.github/.github/workflows/continuous-delivery-dockerfile.yml@main
    secrets: inherit
    with:
      dockerfile-path: "src/MyService/Dockerfile"
      docker-image-name: "my-service-name"
```

---

## Workflow completo de referencia (`.github/workflows/ci.yml`)

```yaml
name: CI/CD

on:
  push:
    branches: [main, master, develop]
    tags:
      - "v*.*.*"
  pull_request:
    branches: [main, master, develop]

jobs:
  conventions:
    if: github.event_name == 'pull_request'
    name: Conventions
    uses: hamekoz/.github/.github/workflows/conventions.yml@main

  continuous-integration:
    name: Continuous Integration
    needs: [conventions]
    if: always() && (needs.conventions.result == 'success' || needs.conventions.result == 'skipped')
    uses: hamekoz/.github/.github/workflows/dotnet.yml@main
    secrets: inherit

  continuous-delivery-nuget:
    name: Publish NuGet
    needs: [continuous-integration]
    if: github.event_name == 'push' && startsWith(github.ref, 'refs/tags/v')
    uses: hamekoz/.github/.github/workflows/continuous-delivery-nuget.yml@main
    secrets: inherit

  continuous-delivery-docker:
    name: Publish Docker
    needs: [continuous-integration]
    if: github.event_name == 'push'
    uses: hamekoz/.github/.github/workflows/continuous-delivery-dockerfile.yml@main
    secrets: inherit
    with:
      docker-image-name: "my-service-name"
```

---

## Requisitos del repositorio .NET

Para que los workflows funcionen correctamente, el repositorio debe tener:

- `global.json` en la raíz con la versión del SDK fijada.
- `GitVersion.yml` (opcional, para configuración avanzada de versionado).
- `nuget.config` si se usan paquetes de GitHub Packages (ver [ejemplo](../../dotnet-examples/nuget.config)).

---

## Cobertura de código

Los reportes de cobertura se suben automáticamente a Codecov. Para habilitarlo:

1. Crear cuenta en [codecov.io](https://codecov.io) y vincular el repositorio.
2. Agregar el secret `CODECOV_TOKEN` en la configuración del repositorio.

El badge de cobertura se puede agregar al `README.md`:

```markdown
[![codecov](https://codecov.io/gh/hamekoz/my-repo/branch/main/graph/badge.svg)](https://codecov.io/gh/hamekoz/my-repo)
```

---

## Version.txt en APIs

El workflow de CD inyecta la versión como build arg de Docker:

```dockerfile
ARG version
ENV SOURCE_VERSION=$version
```

Para exponerlo en el endpoint `/version`:

```csharp
app.MapGet("/version", () => Environment.GetEnvironmentVariable("SOURCE_VERSION") ?? "unknown");
```
