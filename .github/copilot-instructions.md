# Hamekoz — GitHub Copilot Instructions

This file configures GitHub Copilot behavior across all repositories of the Hamekoz organization.
For the full organization criterion, read the org `AGENTS.md`:
`hamekoz/.github/docs/ai/AGENTS.md`.

## Context

- **Primary stack**: .NET / C# (APIs, microservices, workers, NuGet libraries).
- **Secondary stack**: Ruby on Rails.
- **Infrastructure**: Docker, GitHub Actions.

## Commits and branches

- Always use **Conventional Commits**: `<type>(<scope>): <description>`.
- Valid types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`,
  `chore`, `revert` (see `docs/conventions/conventional-commits.md`).
- Title: lowercase, imperative, no trailing period, max 100 characters. English only.
- Branch names use **semantic prefixes by change type** (`feature/`, `fix/`, `chore/`, `docs/`,
  `refactor/`). Never use an agent-specific prefix.

## Language

- Source code (variables, methods, classes, comments): **English**.
- UI strings: `es-AR` with `en` fallback per project.
- Documentation: English for criterion docs (Spanish copies available with `.es.md` suffix).

## Code quality

- **Clean Code** (`docs/architecture/clean-code.md`): small functions, single responsibility,
  descriptive names. Variables are named by **content**, never by type.
- No commented or dead code. No empty `catch`. Structured logging with `{Placeholders}`.
- No secrets or credentials in source. Modify only files related to the task.

## Architecture

- **Clean Architecture** (`docs/architecture/clean-architecture.md`): dependencies point inward
  (Core ← Services ← Data/Delivery).
- Minimal APIs in `Endpoints/`; DI registered only in `Program.cs` (primary constructors).
- Apply **12 Factor App** principles (config, processes, logs).

## .NET (C#)

- `dotnet format --verify-no-changes` must pass.
- Constructor injection; never `new ConcreteService()` in business code.
- Async methods with `Async` suffix; propagate `CancellationToken`.
- Private fields with `_` prefix; constants `UPPER_SNAKE_CASE`.
- New test projects: Microsoft Testing Platform + `xunit.v3` + `coverlet.MTP`
  (`docs/testing/dotnet.md`). Legacy: VSTest + `coverlet.collector`.

## Ruby on Rails

- `bundle exec rubocop` must pass.
- Business logic in service objects, not in controllers/models.
- Tests with RSpec, Arrange/Act/Assert.

## Security

- Never commit secrets, API keys, or tokens. Use GitHub Secrets in CI/CD.
- Validate all external inputs; avoid SQL injection, XSS, SSRF.
- Do not introduce known vulnerable dependencies.

## CI/CD

- Do not disable or bypass CI checks; do not modify org workflows without explicit justification.
- Reusable org workflows live in `hamekoz/.github`.

## # Publishing Workflows

The organization provides reusable workflows for automated publishing:

**NuGet packages**:

- `publish-nuget-to-github-packages.yml` — Publish to GitHub Packages
- `publish-nuget-to-nuget-org.yml` — Publish to NuGet.org
- Optional input `nuget-version` for manual versioning; defaults to GitVersion if not provided

**Docker images**:

- `publish-docker-image-to-github-packages.yml` — Publish to GitHub Container Registry
- `publish-docker-image-to-dockerhub.yml` — Publish to Docker Hub

See [`docs/workflows/publishing.md`](./docs/workflows/publishing.md) for examples and configuration.

## Process

Before generating code:

1. Read the relevant docs and existing code for context.
2. Check that no similar implementation already exists.
3. Declare explicitly which files will be modified.
4. Write tests for new behavior.
5. Run lint and tests after changes.
6. Update the repository AI task log (if present).

When in doubt or when impact is high, ask before acting.
