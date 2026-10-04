# Repository Onboarding Checklist

This checklist applies to every repository in the organization: new ones and existing ones that do
not yet meet the standards.

---

## New repositories

Complete **before the first real functional commit**.

## # Basic structure

- [ ] `README.md` at the root with description, minimal architecture, and contributor guide.
- [ ] Appropriate `.gitignore` for the stack.
- [ ] `.editorconfig` copied from `hamekoz/.github` or adjusted for the stack.
- [ ] `AGENTS.md` at the root with the stub pointing to the organization criterion (see
      [AGENTS.md](../ai/AGENTS.md#repository-guidance)).
- [ ] `CHANGELOG.md` with an initial entry.
- [ ] Set `main` as the default branch.

## # Development environment

- [ ] `.vscode/settings.json` with the base workspace configuration (see
      [VS Code guide](../tooling/vscode.md)).
- [ ] `.vscode/extensions.json` with recommended extensions for the stack.
- [ ] (Recommended) `.devcontainer/devcontainer.json` with a reproducible development environment
      (see [Dev Containers guide](../tooling/devcontainer.md)).
- [ ] `.env.example` with the required environment variables (no real values) if the project uses
      environment variables.

## # Code quality

- [ ] Configure the lint/format tool for the stack:
  - .NET: `global.json` + `dotnet format` configured (see
    [Code Format .NET](../code-format/dotnet.md)).
  - Ruby: `.rubocop.yml` + RuboCop in the Gemfile.
- [ ] Verify lint passes green from the start.

## # Testing

- [ ] New projects: Microsoft Testing Platform stack (`xunit.v3` + `coverlet.MTP`,
      `<UseMicrosoftTestingPlatformRunner>true</UseMicrosoftTestingPlatformRunner>`, and
      `"test": { "runner": "Microsoft.Testing.Platform" }` in `global.json`).
- [ ] Legacy projects: keep VSTest (`xunit` 2.9.x + `coverlet.collector`).
- [ ] At least one example test (even a "hello world") passing in CI.
- [ ] Coverage configured in CI with the criterion thresholds (line ≥ 80%, branch ≥ 70%).
- [ ] The shared `dotnet.yml` workflow is consumed with the `test_arguments` input matching the
      project runner (see [Testing .NET](../testing/dotnet.md#ci--shared-workflow)).

## # CI/CD

- [ ] `.github/workflows/ci.yml` configured with at least:
  - Conventional Commits check on PRs.
  - Format verification.
  - Build + tests.
- [ ] Branch protection configured in GitHub for `main` and `develop`:
  - Require PR before merging.
  - Require status checks to pass.
  - Include administrators.
- [ ] At least one example test (even a "hello world") passing in CI.

## # Versioning

- [ ] Versioning strategy defined (GitVersion for .NET, or manual with tags).
- [ ] First version tag created: `v0.1.0`.

## # Security

- [ ] Confirmed that no secrets or credentials are present in the code.
- [ ] Dependabot or Renovate configured for automatic updates.
- [ ] `.github/renovate.json` or `.github/dependabot.yml` present.

## # Team documentation

- [ ] Repository owner documented in `README.md`.
- [ ] Responsible team or contact identified.

## # AI usage (if applicable)

- [ ] Create `AGENTS.md` at the root with the stub pointing to the organization criterion (see
      [AGENTS.md](../ai/AGENTS.md)).
- [ ] Create `docs/ai/ai-task-log.md` if the repository will use AI agents.
- [ ] Review that the organization `.github/copilot-instructions.md` covers the project context (or
      add a local one with project specifics).

---

## Existing repositories

For repositories that already have history but do not meet all the standards.

## # High priority (week 1)

- [ ] Basic CI pipeline working (build + test).
- [ ] Branch protection on `main`.
- [ ] No secrets exposed in the commit history.

## # Medium priority (month 1)

- [ ] Conventional Commits active on new PRs.
- [ ] Lint/format verified in CI.
- [ ] `README.md` updated with description and basic guide.
- [ ] Dependabot or Renovate enabled.

## # Low priority (quarter 1)

- [ ] CHANGELOG created and updated.
- [ ] Test coverage with report in CI.
- [ ] Semantic versioning with tags.
- [ ] Architecture documented in `README.md` or `docs/`.

---

## Adoption matrix by stack

| Tool/Practice             | .NET | Ruby on Rails | Required    |
| ------------------------- | ---- | ------------- | ----------- |
| Conventional Commits      | ✅   | ✅            | Yes         |
| CI on every PR            | ✅   | ✅            | Yes         |
| Branch protection         | ✅   | ✅            | Yes         |
| dotnet format / RuboCop   | ✅   | ✅            | Yes         |
| Automated tests           | ✅   | ✅            | Yes         |
| Semantic Versioning       | ✅   | ✅            | Yes         |
| `.vscode/settings.json`   | ✅   | ✅            | Recommended |
| `.vscode/extensions.json` | ✅   | ✅            | Recommended |
| `.devcontainer/`          | ✅   | ✅            | Recommended |
| `.env.example`            | ✅   | ✅            | Recommended |
| Codecov / coverage        | ✅   | ✅            | Recommended |
| `AGENTS.md` (org stub)    | ✅   | ✅            | Yes         |
| Brakeman (security)       | N/A  | ✅            | Yes (Rails) |
| bundler-audit             | N/A  | ✅            | Yes (Rails) |
| Dependabot/Renovate       | ✅   | ✅            | Recommended |
| CHANGELOG.md              | ✅   | ✅            | Recommended |
| docs/ai/ai-task-log.md    | ✅   | ✅            | If AI used  |

---

## How to reference organization workflows

To use the shared workflows in a repository:

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

See more details in [CI/CD — General guide](../ci-cd/README.md).
