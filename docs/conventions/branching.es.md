# Estrategia de Branching

## Ramas principales

| Rama      | Propósito                                               | Ambiente destino |
| --------- | ------------------------------------------------------- | ---------------- |
| `main`    | Código en producción. Siempre estable y deployable.     | Producción       |
| `develop` | Integración de features. Base para branches de trabajo. | Integración / QA |

Ambas ramas están **protegidas**: no se puede hacer push directo; todo cambio entra por Pull
Request.

## Ramas de trabajo

Las ramas de trabajo se crean desde `develop` y se eliminan al hacer merge:

| Prefijo     | Propósito                                    | Ejemplo                            |
| ----------- | -------------------------------------------- | ---------------------------------- |
| `feature/`  | Nueva funcionalidad                          | `feature/add-oauth-login`          |
| `fix/`      | Corrección de bug                            | `fix/null-reference-on-checkout`   |
| `hotfix/`   | Corrección urgente en producción             | `hotfix/critical-auth-bypass`      |
| `chore/`    | Mantenimiento, deps, infra                   | `chore/update-sdks`                |
| `docs/`     | Solo documentación                           | `docs/update-api-guide`            |
| `refactor/` | Refactorización sin cambio de comportamiento | `refactor/extract-payment-service` |

**Los agentes IA usan los mismos prefijos semánticos** según el tipo de cambio. No existe un
prefijo específico para agentes.

## Flujo de trabajo estándar

```text
main ◄─── develop ◄─── feature/my-feature
           │
           └──── fix/some-bug
```

1. Crear rama desde `develop`: `git checkout -b feature/my-feature develop`
2. Desarrollar y commitear con Conventional Commits.
3. Abrir Pull Request hacia `develop`.
4. CI pasa (conventional commits + format + build + tests).
5. Code review aprobado (obligatorio para cambios generados por IA, ver
   [política de revisión](../ai/review-policy.md)).
6. Merge a `develop` (squash merge recomendado).
7. Periódicamente, `develop` → `main` via PR con tag de versión.

## Hotfixes en producción

Un hotfix crítico que no puede esperar el ciclo normal:

1. Crear rama desde `main`: `git checkout -b hotfix/critical-fix main`
2. Aplicar fix y tests.
3. Merge a `main` con PR + CI obligatorio.
4. Tag de versión con incremento PATCH.
5. Merge de vuelta a `develop` para sincronizar.

## Ambientes

| Ambiente    | Rama                   | Trigger de deploy         |
| ----------- | ---------------------- | ------------------------- |
| Integración | `develop`              | Push a `develop`          |
| Staging/UAT | `uat`/`stg` (opcional) | Push o tag de pre-release |
| Producción  | `main`                 | Tag `v*.*.*` en `main`    |

## Reglas de protección de ramas

Configurar en GitHub las siguientes branch protection rules para `main` y `develop`:

- ✅ Require pull request before merging
- ✅ Require status checks to pass (CI workflow)
- ✅ Require branches to be up to date before merging
- ✅ Require linear history (opcional, recomendado)
- ✅ Include administrators
- ❌ Allow force pushes
- ❌ Allow deletions
