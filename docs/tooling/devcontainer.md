# Dev Containers

Los Dev Containers permiten definir el entorno de desarrollo completo como código, garantizando que todos los desarrolladores trabajen con las mismas herramientas, versiones y configuración.

Referencia: [containers.dev](https://containers.dev/)

---

## Requisitos previos

| Herramienta                                                                                                        | Versión mínima | Propósito                           |
| ------------------------------------------------------------------------------------------------------------------ | -------------- | ----------------------------------- |
| [Docker Desktop](https://www.docker.com/products/docker-desktop/)                                                  | Última estable | Runtime de contenedores             |
| [VS Code](https://code.visualstudio.com/)                                                                          | Última estable | Editor con soporte Dev Containers   |
| [Dev Containers extension](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) | Última         | Extensión VS Code para devcontainer |

También es posible usar Dev Containers con GitHub Codespaces, sin necesidad de Docker local.

---

## Estructura mínima

Todo repositorio que use Dev Containers debe tener:

```text
. devcontainer/
└── devcontainer.json     ← Configuración principal del contenedor
```

Opcionalmente:

```text
. devcontainer/
├── devcontainer.json
├── Dockerfile            ← Si se necesita una imagen personalizada
└── docker-compose.yml    ← Si el entorno requiere múltiples servicios (BD, cache, etc.)
```

---

## Convenciones

- El `devcontainer.json` se **compromete al repositorio** junto con el código.
- Usar imágenes base oficiales de Microsoft (`mcr.microsoft.com/devcontainers/`) siempre que sea posible.
- Especificar la versión de la imagen, no usar `latest` en producción.
- Agregar las extensiones de VS Code necesarias en `customizations.vscode.extensions` para que se instalen automáticamente.
- Usar `postCreateCommand` para restaurar dependencias y dejar el entorno listo para trabajar.
- Los secrets y variables de entorno sensibles **nunca** van en `devcontainer.json`; usar `.env` local ignorado por `.gitignore` o el mecanismo de secretos de Codespaces.

---

## Devcontainer para proyectos .NET

Ver ejemplo completo en [`devcontainer-examples/dotnet/`](../../devcontainer-examples/dotnet/).

```json
{
  "name": "My .NET Service",
  "image": "mcr.microsoft.com/devcontainers/dotnet:8.0",
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-dotnettools.csdevkit",
        "ms-dotnettools.vscodeintellicode-csharp",
        "editorconfig.editorconfig",
        "streetsidesoftware.code-spell-checker",
        "github.copilot",
        "github.copilot-chat",
        "eamodio.gitlens",
        "davidanson.vscode-markdownlint",
        "github.vscode-github-actions"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "editor.rulers": [110],
        "files.trimTrailingWhitespace": true,
        "files.insertFinalNewline": true
      }
    }
  },
  "postCreateCommand": "dotnet restore",
  "remoteUser": "vscode"
}
```

## # Con base de datos (docker-compose)

Para servicios que requieren PostgreSQL u otros servicios externos:

```json
{
  "name": "My .NET Service with DB",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "ms-dotnettools.csdevkit",
        "editorconfig.editorconfig",
        "github.copilot",
        "github.copilot-chat"
      ]
    }
  },
  "postCreateCommand": "dotnet restore"
}
```

`docker-compose.yml` de ejemplo:

```yaml
services:
  app:
    image: mcr.microsoft.com/devcontainers/dotnet:8.0
    volumes:
      - ../..:/workspaces:cached
    command: sleep infinity

  postgres:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: devpassword
      POSTGRES_DB: myservice_dev
    ports:
      - "5432:5432"
```

---

## Devcontainer para proyectos Ruby on Rails

Ver ejemplo completo en [`devcontainer-examples/ruby-on-rails/`](../../devcontainer-examples/ruby-on-rails/).

```json
{
  "name": "My Rails App",
  "dockerComposeFile": "docker-compose.yml",
  "service": "app",
  "workspaceFolder": "/workspaces/${localWorkspaceFolderBasename}",
  "features": {
    "ghcr.io/devcontainers/features/github-cli:1": {}
  },
  "customizations": {
    "vscode": {
      "extensions": [
        "shopify.ruby-lsp",
        "kaiwood.endwise",
        "editorconfig.editorconfig",
        "streetsidesoftware.code-spell-checker",
        "github.copilot",
        "github.copilot-chat",
        "eamodio.gitlens",
        "davidanson.vscode-markdownlint",
        "github.vscode-github-actions"
      ],
      "settings": {
        "editor.formatOnSave": true,
        "[ruby]": {
          "editor.defaultFormatter": "shopify.ruby-lsp"
        }
      }
    }
  },
  "postCreateCommand": "bundle install && yarn install --frozen-lockfile",
  "remoteUser": "vscode"
}
```

---

## Uso con GitHub Codespaces

Los Dev Containers son totalmente compatibles con [GitHub Codespaces](https://github.com/features/codespaces).
Al presionar el botón "Code → Open with Codespaces" en GitHub, se usa automáticamente el `.devcontainer/devcontainer.json` del repositorio.

Para proyectos de la organización con Codespaces habilitado:

1. El `.devcontainer/devcontainer.json` debe estar en el repositorio.
2. Los secrets de Codespaces se configuran en el perfil de usuario o en la organización (no en el código).
3. Los `postCreateCommand` deben ser idempotentes (seguros de ejecutar más de una vez).

---

## Variables de entorno en Dev Containers

Para variables que varían por desarrollador (tokens, credenciales locales), usar un archivo `.env` local:

```bash
# .env.local (ignorado por .gitignore)
DATABASE_URL=postgres://postgres:devpassword@localhost:5432/myapp_dev
GITHUB_TOKEN=ghp_...
```

Y configurar en `devcontainer.json`:

```json
{
  "runArgs": ["--env-file", ".env.local"]
}
```

> Asegurarse de agregar `.env.local` al `.gitignore`.

---

## Verificación de salud del devcontainer

Al abrir el repositorio en un Dev Container, verificar:

- [ ] El proyecto compila / la aplicación arranca sin errores.
- [ ] Las extensiones de VS Code recomendadas están instaladas.
- [ ] El linter (dotnet format / rubocop) está disponible en la terminal.
- [ ] Los tests corren correctamente desde la terminal.
- [ ] Las variables de entorno necesarias están configuradas.
