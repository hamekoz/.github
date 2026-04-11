# Semantic Versioning

Todos los proyectos de la organización usan [Semantic Versioning 2.0.0](https://semver.org/) para identificar versiones de releases públicos.

## Formato de versión

```text
v<MAJOR>.<MINOR>.<PATCH>
```

Ejemplo: `v2.4.1`

### Reglas de incremento

| Segmento | Cuándo incrementar                                             |
| -------- | -------------------------------------------------------------- |
| `MAJOR`  | Cambio incompatible con versiones anteriores (breaking change) |
| `MINOR`  | Nueva funcionalidad compatible hacia atrás                     |
| `PATCH`  | Corrección de bugs compatible hacia atrás                      |

> Al incrementar `MAJOR`, resetear `MINOR` y `PATCH` a `0`.
> Al incrementar `MINOR`, resetear `PATCH` a `0`.

## Versiones de pre-release

Para artefactos no finales, usar sufijo con identificador:

```text
v1.2.0-alpha.1
v1.2.0-beta.3
v1.2.0-rc.1
```

## Relación con Conventional Commits

La determinación automática de versión se basa en los tipos de commit presentes desde la última versión:

| Commits presentes                        | Incremento |
| ---------------------------------------- | ---------- |
| Al menos un `feat!` o `BREAKING CHANGE`  | MAJOR      |
| Al menos un `feat` (sin breaking change) | MINOR      |
| Solo `fix`, `perf`, `refactor`, etc.     | PATCH      |

## Herramienta de versionado automático

Los proyectos .NET usan [GitVersion](https://gitversion.net/) para calcular la versión automáticamente a partir del historial de commits:

```yaml
- name: Install GitVersion
  uses: gittools/actions/gitversion/setup@v1.2.0
  with:
    versionSpec: "5.x"

- name: Determine Version
  id: version
  uses: gittools/actions/gitversion/execute@v1.2.0
```

## Tags de Git

- Los releases se crean con tags de Git anotados: `git tag -a v1.2.3 -m "Release v1.2.3"`.
- Los tags de release siguen el patrón `v*.*.*`.
- Los tags se crean **únicamente en la rama `main`** después de un merge exitoso.
- Los tags se publican automáticamente como GitHub Releases mediante workflows de CD.

## Version.txt en APIs

Las APIs exponen un endpoint o archivo `version.txt` con el siguiente formato:

```bnf
<valid_version_info> ::= <full_version>
                      | <artifact_repo> ":" <full_version>
                      | <full_version> "@" <source_code_repo_url>
                      | <artifact_repo> ":" <full_version> "@" <source_code_repo_url>

<full_version> ::= <name_or_version> "+" <source_code_commit_id>
                 | <name_or_version> "_" <source_code_commit_id>

<name_or_version> ::= <semver_version>
                    | <name>

<name> ::= "INT" | "main" | "master" | "TEST" | "develop"

<semver_version> ::= "v" <major> "." <minor> "." <patch>

<major> ::= <digits>
<minor> ::= <digits>
<patch> ::= <digits>

<artifact_repo> ::= <docker_hub_org> "/" <docker_hub_repo>

# <digits> matches /\d+/
# Full regex: /(?:(?<artifact_repo>[\w-]+\/[\w-]+):)?(?<version>v\d+\.\d+\.\d+|INT|main|master|TEST|develop)[_+](?<commit>\w+)(?:@(?<repo_url>.+))?/
```

Ejemplo: `v1.4.2+a3f9c12@https://github.com/hamekoz/my-api`

## CHANGELOG

Cada repositorio debe mantener un `CHANGELOG.md` en la raíz que documente los cambios por versión, generado automáticamente a partir de los commits convencionales o actualizado manualmente antes de cada release.

Formato recomendado: [Keep a Changelog](https://keepachangelog.com/).
