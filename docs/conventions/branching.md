# Branching Strategy

## Main branches

| Branch    | Purpose                                         | Target environment |
| --------- | ----------------------------------------------- | ------------------ |
| `main`    | Production code. Always stable and deployable.  | Production         |
| `develop` | Feature integration. Base for working branches. | Integration / QA   |

Both branches are **protected**: no direct push; every change enters through a Pull Request.

## Working branches

Working branches are created from `develop` and deleted after merging:

| Prefix      | Purpose                             | Example                            |
| ----------- | ----------------------------------- | ---------------------------------- |
| `feature/`  | New functionality                   | `feature/add-oauth-login`          |
| `fix/`      | Bug fix                             | `fix/null-reference-on-checkout`   |
| `hotfix/`   | Urgent production fix               | `hotfix/critical-auth-bypass`      |
| `chore/`    | Maintenance, deps, infra            | `chore/update-sdks`                |
| `docs/`     | Documentation only                  | `docs/update-api-guide`            |
| `refactor/` | Refactoring without behavior change | `refactor/extract-payment-service` |

**AI agents use the same semantic prefixes** based on the type of change. There is no
agent-specific prefix.

## Standard workflow

```text
main ◄─── develop ◄─── feature/my-feature
           │
           └──── fix/some-bug
```

1. Create the branch from `develop`: `git checkout -b feature/my-feature develop`
2. Develop and commit with Conventional Commits.
3. Open a Pull Request toward `develop`.
4. CI passes (conventional commits + format + build + tests).
5. Code review approved (mandatory for AI-generated changes, see
   [review policy](../ai/review-policy.md)).
6. Merge to `develop` (squash merge recommended).
7. Periodically merge `develop` → `main` via PR with a version tag.

## Hotfixes in production

A critical fix that cannot wait for the normal cycle:

1. Create the branch from `main`: `git checkout -b hotfix/critical-fix main`
2. Apply the fix and tests.
3. Merge to `main` with PR + mandatory CI.
4. Bump PATCH version with a tag.
5. Merge back to `develop` to synchronize.

## Environments

| Environment | Branch                    | Deploy trigger          |
| ----------- | ------------------------- | ----------------------- |
| Integration | `develop`                 | Push to `develop`       |
| Staging/UAT | `uat` or `stg` (optional) | Push or pre-release tag |
| Production  | `main`                    | Tag `v*.*.*` on `main`  |

## Branch protection rules

Configure these GitHub branch protection rules for `main` and `develop`:

- ✅ Require pull request before merging
- ✅ Require status checks to pass (CI workflow)
- ✅ Require branches to be up to date before merging
- ✅ Require linear history (recommended)
- ✅ Include administrators
- ❌ Allow force pushes
- ❌ Allow deletions
