# Reglas de Formato de Código — Comunes a todos los stacks

Este documento define las reglas de formato aplicables a **todos los proyectos** independientemente del lenguaje. Las reglas específicas por stack se documentan en archivos separados.

Las reglas comunes se configuran y verifican con las herramientas del repositorio `.github` compartido.

---

## Reglas universales (`.editorconfig`)

Configuradas en `.editorconfig` en la raíz de cada repositorio:

| Regla                            | Valor                                                           |
| -------------------------------- | --------------------------------------------------------------- |
| Indentación                      | Espacios (no tabs)                                              |
| Tamaño de indentación            | 2 espacios (excepto donde el lenguaje tiene estándar diferente) |
| Charset                          | UTF-8                                                           |
| Fin de línea                     | LF (`\n`)                                                       |
| Línea vacía al final del archivo | Sí                                                              |
| Espacios al final de línea       | Ninguno                                                         |
| Largo máximo de línea            | 110 caracteres (salvo Markdown y binarios)                      |

---

## Herramientas de verificación

| Herramienta                                                    | Alcance                                       | Config               |
| -------------------------------------------------------------- | --------------------------------------------- | -------------------- |
| [Prettier](https://prettier.io/)                               | HTML, CSS, SCSS, JS, TS, JSON, YAML, Markdown | `.prettierrc`        |
| [EditorConfig Lint (eclint)](https://github.com/jedmao/eclint) | Todos los archivos de texto                   | `.editorconfig`      |
| [ShellCheck](https://www.shellcheck.net/)                      | Scripts `.sh`                                 | Via CI               |
| [markdownlint](https://github.com/DavidAnson/markdownlint)     | Archivos `.md`                                | `.markdownlint.json` |
| [CSpell](https://cspell.org/)                                  | Revisión ortográfica en código                | `cspell.json`        |

---

## Prettier

Configuración base (`.prettierrc`):

```json
{
  "printWidth": 80
}
```

Archivos verificados: `**/*.{html,css,scss,less,js,ts,md,yml,yaml,json}`

---

## Markdown

- Largo de línea: sin límite fijo (facilita diffs en PRs).
- Encabezados deben tener jerarquía correcta (no saltar niveles `h1` → `h3`).
- Usar listas ordenadas (`1.`) solo cuando el orden importa.
- Bloques de código con especificación de lenguaje: ` ```python `.

---

## YAML / JSON

- Indentación: 2 espacios.
- Claves en `kebab-case` para YAML de configuración.
- Sin trailing commas en JSON.

---

## Scripts Shell

- Shebang en primera línea: `#!/usr/bin/env bash`.
- `set -euo pipefail` al inicio de cada script.
- Verificado con ShellCheck; errores SC1091 y SC1090 ignorados por convención de sourcing.

---

## Verificación local

```sh
# Instalar dependencias
yarn install

# Verificar formato (Prettier + EditorConfig)
yarn verify-format

# Verificar Markdown
yarn verify-markdown

# Verificar ortografía
yarn verify-spell

# Corregir formato automáticamente
yarn fix-format
```

---

## Verificación automática en CI

El workflow `conventions.yml` ejecuta automáticamente:

1. Lint de Conventional Commits en todos los commits del PR.
2. Verificación de formato con ShellCheck y Prettier.

Ver más detalles en [CI/CD — Guía general](../ci-cd/README.md).
