# VS Code — Configuración y Extensiones

Visual Studio Code es el editor recomendado para todos los proyectos de la organización. Esta guía define la configuración base y las extensiones sugeridas por stack.

---

## Configuración del workspace (`.vscode/settings.json`)

Incluir este archivo en todos los repositorios para que todos los contribuidores usen la misma configuración base.

```json
{
  "editor.formatOnSave": true,
  "editor.trimAutoWhitespace": true,
  "editor.rulers": [110],
  "files.trimTrailingWhitespace": true,
  "files.insertFinalNewline": true,
  "files.eol": "\n",
  "editor.tabSize": 2,
  "editor.insertSpaces": true,
  "editor.detectIndentation": false,
  "cSpell.language": "en,es",
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[markdown]": {
    "editor.wordWrap": "on"
  }
}
```

### Configuración adicional para .NET

```json
{
  "[csharp]": {
    "editor.defaultFormatter": "ms-dotnettools.csharp",
    "editor.formatOnSave": true
  },
  "dotnet.defaultSolution": "MyService.sln",
  "omnisharp.enableEditorConfigSupport": true,
  "omnisharp.enableRoslynAnalyzers": true
}
```

### Configuración adicional para Ruby on Rails

```json
{
  "[ruby]": {
    "editor.defaultFormatter": "shopify.ruby-lsp",
    "editor.formatOnSave": true
  },
  "rubyLsp.rubyVersionManager": {
    "identifier": "rbenv"
  }
}
```

---

## Extensiones recomendadas (`.vscode/extensions.json`)

Incluir este archivo en todos los repositorios. VS Code mostrará un aviso para instalar las extensiones recomendadas al abrir el proyecto.

### Extensiones generales (todos los proyectos)

```json
{
  "recommendations": [
    "editorconfig.editorconfig",
    "streetsidesoftware.code-spell-checker",
    "streetsidesoftware.code-spell-checker-spanish",
    "github.copilot",
    "github.copilot-chat",
    "eamodio.gitlens",
    "mhutchie.git-graph",
    "ms-azuretools.vscode-docker",
    "davidanson.vscode-markdownlint",
    "esbenp.prettier-vscode",
    "github.vscode-github-actions",
    "ms-vsliveshare.vsliveshare"
  ]
}
```

### Extensiones para .NET (C#)

Agregar a `recommendations`:

```json
[
  "ms-dotnettools.csdevkit",
  "ms-dotnettools.csharp",
  "ms-dotnettools.vscodeintellicode-csharp",
  "formulahendry.dotnet-test-explorer",
  "sonarsource.sonarlint-vscode"
]
```

| Extensión                                 | Descripción                                       |
| ----------------------------------------- | ------------------------------------------------- |
| `ms-dotnettools.csdevkit`                 | C# Dev Kit — suite completa para .NET en VS Code  |
| `ms-dotnettools.csharp`                   | Soporte base de C# (incluido en Dev Kit)          |
| `ms-dotnettools.vscodeintellicode-csharp` | IntelliCode: sugerencias de código con IA para C# |
| `formulahendry.dotnet-test-explorer`      | Explorador de tests para proyectos .NET           |
| `sonarsource.sonarlint-vscode`            | Análisis estático en tiempo real (SonarLint)      |

### Extensiones para Ruby on Rails

Agregar a `recommendations`:

```json
[
  "shopify.ruby-lsp",
  "kaiwood.endwise",
  "connorshea.vscode-ruby-test-adapter",
  "sonarsource.sonarlint-vscode"
]
```

| Extensión                             | Descripción                                                       |
| ------------------------------------- | ----------------------------------------------------------------- |
| `shopify.ruby-lsp`                    | Ruby LSP — soporte completo de Ruby (linting, format, navegación) |
| `kaiwood.endwise`                     | Cierre automático de bloques `do...end`, `if...end`, etc.         |
| `connorshea.vscode-ruby-test-adapter` | Explorador de tests RSpec integrado en VS Code                    |
| `sonarsource.sonarlint-vscode`        | Análisis estático en tiempo real (SonarLint)                      |

---

## Detalle de extensiones clave

### EditorConfig (`editorconfig.editorconfig`)

Aplica automáticamente las reglas del `.editorconfig` al guardar. **Obligatorio** ya que es la fuente de verdad de reglas de formato compartidas.

### CSpell (`streetsidesoftware.code-spell-checker`)

Revisión ortográfica en el código fuente. Las palabras técnicas del proyecto se agregan a `cspell-project-words.txt`.

Habilitar el diccionario español:

```json
{
  "cSpell.language": "en,es",
  "cSpell.userWords": []
}
```

### GitHub Copilot (`github.copilot` + `github.copilot-chat`)

Asistente de IA para autocompletado de código y chat. Requiere licencia de GitHub Copilot activa.

Las instrucciones organizacionales para Copilot están centralizadas en `.github/copilot-instructions.md` y se aplican automáticamente a todos los repositorios de la organización.

Ver [instrucciones para agentes IA](../ai/agent-instructions.md) para el contexto completo.

### GitLens (`eamodio.gitlens`)

Anotaciones de blame, historial de archivos y líneas, comparación de ramas. Recomendado para navegar el historial de cambios.

### GitHub Actions (`github.vscode-github-actions`)

Autocompletado, validación y ejecución de workflows de GitHub Actions directamente desde VS Code.

### Markdownlint (`davidanson.vscode-markdownlint`)

Verifica que los archivos Markdown cumplan con las reglas definidas en `.markdownlint.json`. Complementa el check automático de CI.

---

## Estructura `.vscode/` en cada repositorio

```text
.vscode/
├── settings.json      ← Configuración del workspace (comprometer al repo)
├── extensions.json    ← Extensiones recomendadas (comprometer al repo)
└── launch.json        ← Configuración de debug (comprometer al repo si es útil)
```

> **Nota**: No comprometer `tasks.json` con tareas muy específicas del entorno local. Si se agregan tareas compartidas, documentar su propósito en comentarios.

### Qué comprometer al repo

| Archivo                   | ¿Comprometer?  | Motivo                                                       |
| ------------------------- | -------------- | ------------------------------------------------------------ |
| `.vscode/settings.json`   | ✅ Sí          | Garantiza consistencia entre desarrolladores                 |
| `.vscode/extensions.json` | ✅ Sí          | Facilita el setup inicial                                    |
| `.vscode/launch.json`     | ✅ Si es útil  | Configuraciones de debug reutilizables por el equipo         |
| `.vscode/tasks.json`      | ⚠️ Con cautela | Solo si las tareas son compartidas y no tienen paths locales |

---

## Snippets de equipo

Para snippets de código compartidos, crear `.vscode/snippets/` con archivos JSON por lenguaje y comprometerlos al repositorio.

Ejemplo para C# (`.vscode/snippets/csharp.json`):

```json
{
  "Constructor with logger": {
    "prefix": "ctor-log",
    "body": [
      "private readonly ILogger<${1:ClassName}> _logger;",
      "",
      "public ${1:ClassName}(ILogger<${1:ClassName}> logger)",
      "{",
      "    _logger = logger;",
      "}"
    ],
    "description": "Constructor with ILogger injection"
  }
}
```
