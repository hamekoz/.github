# Checklist de Onboarding de Repositorios

Este checklist debe completarse para todos los repositorios de la organización: tanto nuevos como existentes que aún no cumplan con los estándares.

---

## Repositorios nuevos

Completar **antes del primer commit real de funcionalidad**.

### Estructura básica

- [ ] `README.md` en la raíz con descripción, arquitectura mínima y guía para contribuidores.
- [ ] `.gitignore` apropiado para el stack.
- [ ] `.editorconfig` copiado desde `hamekoz/.github` o ajustado para el stack.
- [ ] `CHANGELOG.md` con entrada inicial.
- [ ] Definir `main` como rama por defecto.

### Entorno de desarrollo

- [ ] `.vscode/settings.json` con configuración base del workspace (ver [guía VS Code](../tooling/vscode.md)).
- [ ] `.vscode/extensions.json` con extensiones recomendadas para el stack.
- [ ] (Recomendado) `.devcontainer/devcontainer.json` con el entorno de desarrollo reproducible (ver [guía Dev Containers](../tooling/devcontainer.md)).
- [ ] `.env.example` con las variables de entorno requeridas (sin valores reales) si el proyecto usa variables de entorno.

### Calidad de código

- [ ] Configurar herramienta de lint/formato para el stack:
  - .NET: `global.json` + `dotnet format` configurado.
  - Ruby: `.rubocop.yml` + `RuboCop` en Gemfile.
- [ ] Verificar que el lint pasa en verde desde el inicio.

### CI/CD

- [ ] `.github/workflows/ci.yml` configurado con al menos:
  - Conventional Commits check en PRs.
  - Verificación de formato.
  - Build + tests.
- [ ] Protección de ramas configurada en GitHub para `main` y `develop`:
  - Require PR before merging.
  - Require status checks to pass.
  - Include administrators.
- [ ] Al menos un test de ejemplo (aunque sea un "hello world") pasando en CI.

### Versionado

- [ ] Estrategia de versionado definida (GitVersion para .NET, o manual con tags).
- [ ] Primer tag de versión creado: `v0.1.0`.

### Seguridad

- [ ] Confirmado que no hay secrets ni credenciales en el código.
- [ ] Dependabot o Renovate configurado para actualizaciones automáticas.
- [ ] `.github/renovate.json` o `.github/dependabot.yml` presente.

### Documentación del equipo

- [ ] Dueño del repositorio documentado en `README.md`.
- [ ] Contacto o equipo responsable identificado.

### Uso de IA (si aplica)

- [ ] Crear `docs/ai/ai-task-log.md` si el repositorio usará agentes IA.
- [ ] Revisar que `.github/copilot-instructions.md` de la organización cubre el contexto del proyecto (o agregar uno local con especificidades del proyecto).

---

## Repositorios existentes

Para repositorios que ya tienen historia pero no cumplen todos los estándares.

### Prioridad alta (semana 1)

- [ ] CI pipeline básico funcionando (build + test).
- [ ] Branch protection en `main`.
- [ ] No hay secrets expuestos en el historial de commits.

### Prioridad media (mes 1)

- [ ] Conventional Commits activo en nuevos PRs.
- [ ] Lint/formato verificado en CI.
- [ ] `README.md` actualizado con descripción y guía básica.
- [ ] Dependabot o Renovate activado.

### Prioridad baja (trimestre 1)

- [ ] CHANGELOG creado y actualizado.
- [ ] Cobertura de tests con reporte en CI.
- [ ] Versionado semántico con tags.
- [ ] Arquitectura documentada en `README.md` o `docs/`.

---

## Matriz de adopción por stack

| Herramienta/Práctica      | .NET | Ruby on Rails | Obligatorio |
| ------------------------- | ---- | ------------- | ----------- |
| Conventional Commits      | ✅   | ✅            | Sí          |
| CI en cada PR             | ✅   | ✅            | Sí          |
| Branch protection         | ✅   | ✅            | Sí          |
| dotnet format / RuboCop   | ✅   | ✅            | Sí          |
| Tests automáticos         | ✅   | ✅            | Sí          |
| Semantic Versioning       | ✅   | ✅            | Sí          |
| `.vscode/settings.json`   | ✅   | ✅            | Recomendado |
| `.vscode/extensions.json` | ✅   | ✅            | Recomendado |
| `.devcontainer/`          | ✅   | ✅            | Recomendado |
| `.env.example`            | ✅   | ✅            | Recomendado |
| Codecov / cobertura       | ✅   | ✅            | Recomendado |
| Brakeman (security)       | N/A  | ✅            | Sí (Rails)  |
| bundler-audit             | N/A  | ✅            | Sí (Rails)  |
| Dependabot/Renovate       | ✅   | ✅            | Recomendado |
| CHANGELOG.md              | ✅   | ✅            | Recomendado |
| docs/ai/ai-task-log.md    | ✅   | ✅            | Si usa IA   |

---

## Cómo referenciar los workflows de la organización

Para usar los workflows compartidos en un repositorio:

```yaml
# .github/workflows/ci.yml
jobs:
  conventions:
    uses: hamekoz/.github/.github/workflows/conventions.yml@main

  dotnet:
    needs: conventions
    uses: hamekoz/.github/.github/workflows/dotnet.yml@main
    secrets: inherit
```

Ver más detalles en [CI/CD — Guía general](../ci-cd/README.md).
