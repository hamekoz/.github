# Hamekoz — AGENTS.md

> **Single source of truth** for AI agents (GitHub Copilot, Gemini, Claude, Cursor, opencode, and
> any OpenAI-compatible coding agent) working in **any** repository of the Hamekoz organization.

This file is the authoritative, organization-wide entry point. It **links to** (it does not
duplicate) the detailed criterion documents.

---

## Repository guidance

Every repository SHOULD create its own `AGENTS.md` (English) and point to this file with a
one-line stub:

```markdown
All instructions from the organization apply. Read them at:
https://github.com/hamekoz/.github/blob/main/docs/ai/AGENTS.md
```

If a repository keeps its own `AGENTS.md`, an entry-point symlink pattern keeps a single source of
truth: `CLAUDE.md`, `AI.md`, `.instructions.md`, `.agents/custom-instructions.md`, and
`.github/copilot-instructions.md` all point to the same `AGENTS.md`.

---

## Organization context

- **Primary stack**: .NET / C# — APIs, microservices, workers, NuGet libraries.
- **Secondary stack**: Ruby on Rails.
- **Infrastructure**: Docker, GitHub Actions.
- **Conventions**: centralized in `hamekoz/.github` (Conventional Commits, SemVer, CI/CD,
  architecture, code, testing, AI policy).

---

## Core rules — all agents

1. **Never commit directly to `main` or `develop`.** All changes go through a branch and a PR.
2. **Follow Conventional Commits** (`<type>(<scope>): <description>`), English, lowercase,
   imperative, title ≤ 100 characters.
3. **Use semantic branch prefixes** (`feature/`, `fix/`, `chore/`, `docs/`, `refactor/`,
   `hotfix/`) according to the type of change. Do NOT invent agent-specific prefixes.
4. **Code is written in English** (identifiers, members, comments). Documentation follows the
   repository convention (criterion docs are English with Spanish copies).
5. **Run lint and tests after meaningful changes.** It must pass before opening a PR.
6. **No secrets** in source code, configs, or commit history.
7. **Atomic commits** — one logical concept per commit (100–200 lines recommended).
8. **Minimal, focused changes.** Do not modify unrelated files or code.
9. **Documentation lives in `docs/`.** Link to criterion docs instead of repeating conventions.
10. **AI task log**: update the repository's task log (if present) after meaningful milestones.

---

## Commit convention

```
<type>[scope]: <description>
```

| Type       | When to use                      |
| ---------- | -------------------------------- |
| `feat`     | New feature                      |
| `fix`      | Bug fix                          |
| `docs`     | Documentation only               |
| `style`    | Formatting / whitespace no logic |
| `refactor` | No behavior change               |
| `perf`     | Performance                      |
| `test`     | Adding/changing tests            |
| `chore`    | Maintenance, deps, tooling       |
| `ci`       | CI/CD configuration              |
| `build`    | Build system                     |

Description: lowercase, imperative mood ("add", not "added"), no trailing period, English only.
CI enforces this with gitlint.

---

## Branching

| Prefix      | Purpose               |
| ----------- | --------------------- |
| `feature/`  | New functionality     |
| `fix/`      | Bug fixes             |
| `hotfix/`   | Urgent production fix |
| `chore/`    | Maintenance           |
| `docs/`     | Documentation         |
| `refactor/` | Restructuring         |

Branches are created from `develop` (or `main` for hotfixes) and merged via PR. See
[Branching strategy](../conventions/branching.md).

---

## Criterion documents (read before coding)

| Dimension          | Document                                                    |
| ------------------ | ----------------------------------------------------------- |
| Architecture       | [Clean Architecture](../architecture/clean-architecture.md) |
| Code quality       | [Clean Code](../architecture/clean-code.md)                 |
| Code format `.NET` | [Code Format .NET](../code-format/dotnet.md)                |
| Testing `.NET`     | [Testing .NET](../testing/dotnet.md)                        |
| AI + human review  | [Review policy](../ai/review-policy.md)                     |

### Highlights

- **Clean Architecture**: dependencies point inward (Core ← Services ← Data/Delivery). Order
  applies. Endpoints in `Endpoints/`, DI only in `Program.cs`.
- **Clean Code**: variables named by content, never by type. Constant literals and `null` use
  named arguments. No empty `catch`. Structured logging with `{Placeholders}`.
- **.NET**: `net10.0`, `Nullable`, `ImplicitUsings`, `LangVersion=latest`,
  `EnforceCodeStyleInBuild`, `AnalysisLevel=latest-Default`, `TreatWarningsAsErrors=false`,
  Central Package Management. `dotnet format --verify-no-changes` must pass.
- **Testing**: xUnit. New projects use Microsoft Testing Platform (`xunit.v3` + `coverlet.MTP`);
  legacy projects keep VSTest. Test doubles are hand-written `Fake*`/`Stub*`. Coverage: line
  ≥ 80%, branch ≥ 70%.

---

## Code style basics (recap)

| Element                        | Convention         | Example           |
| ------------------------------ | ------------------ | ----------------- |
| Classes / Methods / Properties | `PascalCase`       | `GetOrderAsync`   |
| Private fields                 | `_camelCase`       | `_repository`     |
| Constants                      | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Async methods                  | `Async` suffix     | `GetOrderAsync`   |
| Parameters / locals            | `camelCase`        | `orderId`         |

Database models in `es-AR` domain language must keep English identifiers and map
Argentina-specific concepts into explicit value objects with clear names.

---

## Working process

1. Read this file and the relevant criterion document before starting.
2. Read the existing code to understand current context; check for an existing implementation.
3. Declare explicitly which files will be modified.
4. Write tests for new behavior (one failing test first, or a regression test).
5. Run format, build, and tests.
6. Open a PR with a Conventional Commits title; a human must review and approve it.

---

## Security

- Never commit API keys, passwords, connection strings, OAuth credentials, or private keys.
- Use `dotnet user-secrets` for local development; environment variables/GitHub Secrets for the rest.
- Validate every external input; avoid SQL injection, XSS, SSRF.
- Do not disable or bypass CI checks.
- Only humans merge to protected branches; agents never approve their own PRs.

---

_This file is owned by the Hamekoz organization. Update it when the organization criterion
changes; per-repository `AGENTS.md` files remain lightweight pointers._
