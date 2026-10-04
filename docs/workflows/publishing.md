# Workflows de Publicación — NuGet y Docker

Este documento describe los workflows reutilizables para publicar paquetes NuGet e imágenes Docker.

## 📋 Tabla de contenidos

- [Workflows de NuGet](#workflows-de-nuget)
- [Workflows de Docker](#workflows-de-docker)
- [Cómo usar](#cómo-usar)
- [Ejemplos](#ejemplos)

## Workflows de NuGet

### Estructura

- **`continuous-delivery-nuget.yml`** (coordinador) — Orquesta ambos destinos
- **`publish-nuget-to-github-packages.yml`** — Publica en GitHub Packages
- **`publish-nuget-to-nuget-org.yml`** — Publica en NuGet.org

### Características

✅ **Versión configurable**: Input opcional `nuget-version`
- Sin input → determina automáticamente con GitVersion
- Con input → usa versión especificada (más rápido)

✅ **Independientes**: Cada workflow puede invocarse por separado

✅ **Reutilizables**: Invocables desde otros workflows

### Inputs

```yaml
inputs:
  runner:
    description: "Runner donde ejecutar (default: ubuntu-latest)"
    required: false
    default: "ubuntu-latest"
    type: string
  nuget-version:
    description: "Versión del NuGet (default: auto-detect con GitVersion)"
    required: false
    default: ""
    type: string
```

## Workflows de Docker

### Estructura

- **`continuous-delivery-dockerfile.yml`** (coordinador) — Orquesta ambos destinos
- **`publish-docker-image-to-github-packages.yml`** — Publica en GitHub Container Registry
- **`publish-docker-image-to-dockerhub.yml`** — Publica en Docker Hub

### Características

✅ **Multi-proyecto**: Soporta múltiples Dockerfiles y nombres de imagen

✅ **Independientes**: Cada workflow puede invocarse por separado

✅ **Reutilizables**: Invocables desde otros workflows

### Inputs

```yaml
inputs:
  runner:
    description: "Runner donde ejecutar (default: ubuntu-latest)"
    required: false
    default: "ubuntu-latest"
    type: string
  dockerfile-path:
    description: "Path al Dockerfile (default: Dockerfile)"
    default: "Dockerfile"
    type: string
  docker-image-name:
    description: "Nombre de la imagen Docker"
    required: true
    type: string
```

## Cómo usar

### Opción A: Publicación automática en branches

**En tu repositorio**, crear `.github/workflows/cd.yml`:

```yaml
name: Continuous Delivery

on:
  push:
    branches: [main, uat, stg]

jobs:
  publish-nuget:
    uses: hamekoz/.github/.github/workflows/continuous-delivery-nuget.yml@main
    with:
      publish-nuget-org: true  # false para solo GitHub Packages
    secrets: inherit

  publish-docker:
    uses: hamekoz/.github/.github/workflows/continuous-delivery-dockerfile.yml@main
    with:
      dockerfile-path: "DefaultProject/Dockerfile"
      docker-image-name: "my-package"
    secrets: inherit
```

### Opción B: Publicación automática en tags

El template `hamekoz.yml` ya incluye esta configuración:

```yaml
uses: hamekoz/.github/.github/workflows/hamekoz.yml@main
```

Cuando se crea un tag `v1.2.3`, publica automáticamente:
- NuGet v1.2.3 a GitHub Packages
- NuGet v1.2.3 a NuGet.org
- Docker con tag `v1.2.3` a ambos registros

### Opción C: Publicación selectiva a un destino

**Solo GitHub Packages**:
```yaml
publish-nuget-github:
  uses: hamekoz/.github/.github/workflows/publish-nuget-to-github-packages.yml@main
  with:
    nuget-version: "1.2.3"  # opcional, si no está usa GitVersion
  secrets: inherit
```

**Solo Docker Hub**:
```yaml
publish-docker-hub:
  uses: hamekoz/.github/.github/workflows/publish-docker-image-to-dockerhub.yml@main
  with:
    dockerfile-path: "MyProject/Dockerfile"
    docker-image-name: "my-image"
  secrets: inherit
```

### Opción D: Versión manual de NuGet

Para especificar una versión exacta en lugar de usar GitVersion:

```yaml
publish-nuget:
  uses: hamekoz/.github/.github/workflows/continuous-delivery-nuget.yml@main
  with:
    nuget-version: "2.0.0-beta.1"  # versión exacta
    publish-nuget-org: true
  secrets: inherit
```

## Ejemplos

### Ejemplo 1: Repo con un proyecto .NET

```yaml
# .github/workflows/release.yml
name: Release

on:
  push:
    tags:
      - v*.*.*

jobs:
  publish-nuget:
    name: Publish NuGet Package
    uses: hamekoz/.github/.github/workflows/continuous-delivery-nuget.yml@main
    with:
      nuget-version: ${{ github.ref_name }}  # v1.2.3
      publish-nuget-org: true
    secrets: inherit
```

**Result**: Al hacer `git tag v1.2.3 && git push origin v1.2.3`:
- Se publica NuGet 1.2.3 a GitHub Packages
- Se publica NuGet 1.2.3 a NuGet.org

### Ejemplo 2: Repo con múltiples Docker projects

```yaml
# .github/workflows/docker.yml
name: Docker Build

on:
  push:
    branches: [main, develop]

jobs:
  publish-api:
    name: Publish API Docker Image
    uses: hamekoz/.github/.github/workflows/continuous-delivery-dockerfile.yml@main
    with:
      dockerfile-path: "Api/Dockerfile"
      docker-image-name: "my-api"
    secrets: inherit

  publish-worker:
    name: Publish Worker Docker Image
    uses: hamekoz/.github/.github/workflows/continuous-delivery-dockerfile.yml@main
    with:
      dockerfile-path: "Worker/Dockerfile"
      docker-image-name: "my-worker"
    secrets: inherit
```

**Result**: Ambas imágenes se publican en paralelo a:
- GitHub Container Registry (ghcr.io)
- Docker Hub

### Ejemplo 3: Rama de staging con versión específica

```yaml
jobs:
  publish-staged:
    if: github.ref == 'refs/heads/stg'
    name: Publish Staging Release
    uses: hamekoz/.github/.github/workflows/continuous-delivery-nuget.yml@main
    with:
      nuget-version: "1.2.3-stg.${{ github.run_number }}"
      publish-nuget-org: false  # Solo GitHub Packages
    secrets: inherit
```

**Result**: NuGet `1.2.3-stg.42` publicado solo a GitHub Packages

## Secretos requeridos

### Para publicar en GitHub Packages

✅ `GITHUB_TOKEN` — Automático, no requiere configuración

### Para publicar en NuGet.org

✅ `NUGET_API_KEY` — Token de autenticación de NuGet.org
```
Crear en: https://www.nuget.org/account/apikeys
```

### Para publicar en Docker Hub

✅ `DOCKERHUB_USERNAME` — Tu usuario de Docker Hub
✅ `DOCKERHUB_TOKEN` — Personal access token de Docker Hub
```
Crear en: https://hub.docker.com/settings/security
```

## Resolución de problemas

### GitVersion no funciona

**Causa**: No hay commits con tags semver previos

**Solución**: Asegúrate que los commits sean accesibles:
```bash
git fetch --unshallow  # Si es shallow clone
```

### Versión no se refleja en el paquete

**Causa**: Es necesario especificar la versión en invocación

**Solución**:
```yaml
with:
  nuget-version: "1.2.3"  # Especifica versión
```

### Docker image no se publica a Docker Hub

**Causa**: Faltan secretos `DOCKERHUB_USERNAME` o `DOCKERHUB_TOKEN`

**Solución**: Agregar secrets en Settings → Secrets and variables → Actions

## Véase también

- [Semantic Versioning](../../docs/conventions/semantic-versioning.md)
- [Conventional Commits](../../docs/conventions/conventional-commits.md)
- [CI/CD](../../docs/ci-cd/README.md)
