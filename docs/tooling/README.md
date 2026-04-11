# Herramientas de Desarrollo

Este directorio centraliza las convenciones y configuraciones recomendadas para los entornos de desarrollo en la organización Hamekoz.

## Índice

| Documento                           | Descripción                                           |
| ----------------------------------- | ----------------------------------------------------- |
| [VS Code](./vscode.md)              | Configuración, extensiones y settings recomendados    |
| [Dev Containers](./devcontainer.md) | Entornos de desarrollo reproducibles con devcontainer |

## Principio rector

> **Un entorno de desarrollo reproducible elimina el "en mi máquina funciona".**
> El objetivo es que cualquier desarrollador pueda clonar un repositorio y comenzar a trabajar en minutos, con las mismas herramientas y configuración que el resto del equipo.

## Herramientas principales

| Herramienta                                                       | Propósito                                     | Obligatorio |
| ----------------------------------------------------------------- | --------------------------------------------- | ----------- |
| [VS Code](https://code.visualstudio.com/)                         | Editor principal                              | Recomendado |
| [Dev Containers](https://containers.dev/)                         | Entorno reproducible vía Docker               | Recomendado |
| [GitHub CLI (`gh`)](https://cli.github.com/)                      | Gestión de PRs e issues desde terminal        | Recomendado |
| [Docker Desktop](https://www.docker.com/products/docker-desktop/) | Runtime de contenedores local                 | Recomendado |
| [EditorConfig](https://editorconfig.org/)                         | Reglas de formato consistentes entre editores | Obligatorio |

## Configuraciones de ejemplo

Los archivos de ejemplo para Dev Containers están disponibles en el directorio [`devcontainer-examples/`](../../devcontainer-examples/):

- [`devcontainer-examples/dotnet/`](../../devcontainer-examples/dotnet/) — para proyectos .NET
- [`devcontainer-examples/ruby-on-rails/`](../../devcontainer-examples/ruby-on-rails/) — para proyectos Ruby on Rails
