# Hamekoz — Estándares y Convenciones Organizacionales

Este directorio es el **hub centralizado** de estándares, convenciones y políticas que aplican a **todos los repositorios** de la organización Hamekoz.

## Índice

## # Convenciones de desarrollo

| Documento                                                     | Descripción                                 |
| ------------------------------------------------------------- | ------------------------------------------- |
| [Conventional Commits](./conventions/conventional-commits.md) | Formato obligatorio de mensajes de commit   |
| [Semantic Versioning](./conventions/semantic-versioning.md)   | Política de versionado semántico y releases |
| [Branching](./conventions/branching.md)                       | Estrategia de ramas y ambientes             |

## # Arquitectura y diseño

| Documento                                                  | Descripción                                  |
| ---------------------------------------------------------- | -------------------------------------------- |
| [12 Factor](./architecture/12-factor.md)                   | Checklist 12 Factor App por tipo de servicio |
| [Clean Code](./architecture/clean-code.md)                 | Criterios de código limpio                   |
| [Clean Architecture](./architecture/clean-architecture.md) | Organización en capas y dependencias         |
| [Microservicios](./architecture/microservices.md)          | Principios y límites de servicios            |

## # Formato y calidad de código

| Documento                                       | Descripción                                     |
| ----------------------------------------------- | ----------------------------------------------- |
| [Reglas comunes](./code-format/README.md)       | Reglas de formato aplicables a todos los stacks |
| [.NET](./code-format/dotnet.md)                 | Convenciones específicas .NET / C#              |
| [Ruby on Rails](./code-format/ruby-on-rails.md) | Convenciones específicas Ruby on Rails          |
| [.editorconfig](./.editorconfig)                | Reglas de formato de la organización            |

## # Testing

| Documento                   | Descripción                                   |
| --------------------------- | --------------------------------------------- |
| [.NET](./testing/dotnet.md) | Stack de testing (MTP/VSTest), cobertura y CI |

## # CI/CD

| Documento                                 | Descripción                              |
| ----------------------------------------- | ---------------------------------------- |
| [Guía general](./ci-cd/README.md)         | Pipeline mínimo por tipo de proyecto     |
| [.NET](./ci-cd/dotnet.md)                 | Workflows reutilizables para .NET        |
| [Ruby on Rails](./ci-cd/ruby-on-rails.md) | Workflows y pipelines para Ruby on Rails |

## # Inteligencia Artificial

| Documento                                                   | Descripción                                             |
| ----------------------------------------------------------- | ------------------------------------------------------- |
| [AGENTS.md](./ai/AGENTS.md)                                 | Fuente única del criterio para agentes IA (inglés)      |
| [Instrucciones para agentes IA](./ai/agent-instructions.md) | Política y contexto para Copilot y otros agentes        |
| [Política de revisión](./ai/review-policy.md)               | Revisión humana obligatoria de cambios generados por IA |
| [Plantilla de tarea IA](./ai/task-template.md)              | Estructura estándar para solicitar trabajo a un agente  |
| [Plantilla de historial IA](./ai/ai-task-log-template.md)   | Formato de registro de tareas realizadas con IA         |

## # Herramientas de desarrollo

| Documento                                   | Descripción                                        |
| ------------------------------------------- | -------------------------------------------------- |
| [VS Code](./tooling/vscode.md)              | Configuración, settings y extensiones recomendadas |
| [Dev Containers](./tooling/devcontainer.md) | Entornos de desarrollo reproducibles               |

## # Adopción y onboarding

| Documento                                                     | Descripción                                                 |
| ------------------------------------------------------------- | ----------------------------------------------------------- |
| [Checklist de onboarding](./adoption/onboarding-checklist.md) | Lista de verificación para repositorios nuevos y existentes |

---

## Principio rector

> **"Core común organizacional + anexos por stack."**
> Las reglas del núcleo aplican a todos los proyectos. Las extensiones por stack (.NET, Rails, etc.) complementan sin contradecir.

## Idioma de los documentos

Los documentos de criterio se escriben en **inglés (canónico)** y se publican con una copia en
**español** con sufijo `.es.md` en el mismo directorio (p. ej. `clean-code.md` + `clean-code.es.md`).
Los archivos consumidos por agentes IA (`AGENTS.md`, `copilot-instructions.md`) se mantienen solo
en inglés.

## Cómo contribuir a este hub

1. Abrir un PR con la propuesta de cambio en el documento correspondiente.
2. Usar Conventional Commits en el título del PR.
3. Obtener al menos una aprobación de un maintainer.
4. Los cambios en este repositorio se propagan automáticamente a todos los proyectos que consumen los workflows compartidos.
