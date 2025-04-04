# Hamekoz Projects

## Minimal requirements for new projects

- Only choose private repositories when the code has business value differentiator logic.

- Use [Hamekoz Organization](https://github.com/hamekoz/) for public repositories

- Do not store secrets in the repository

- Add a `README.md` file with:

  - Basic description of the project scope

  - Minimal architecture design

  - Guide for contributors

  - Coding standard (When it is not covered by an automated CI process)

- CI/CD from the beginning

  - Setup CI/CD and all environments even before adding any real functionality

  - Branch protection rules to avoid CI bypass

  - Example automated test even before adding any real functionality

## Nice to have

- Consider base your repository in one of our base projects [See more](#base-projects)

- Validate coding standards using automated CI process [See more](#ci-cd)

- Follow Doppler convention for branches and environments [See more](#ci-cd)

- Generate a version.txt file, or another simple way to check what is the code related to the published system. See more details below. [See more](#version-txt)

- Authenticate users using JWT with our public key [See more]{#doppler-security}

### Version txt

Knowing exactly what source code corresponds to the system we are running use to be very useful.

In our APIs projects we expose a `version.txt` for that

The version format is the following:

```bnf
<valid_version_info> ::= <full_version>
                      | <artifact_repo> ":" <full_version>
                      | <full_version> "@" <source_code_repo_url>
                      | <artifact_repo> ":" <full_version> "@" <source_code_repo_url>

<full_version> ::= <name_or_version> "+" <source_code_commit_id>
                | <name_or_version> "_" <source_code_commit_id>

<name_or_version> ::= <semver_version>
                    | <name>

<name> ::= "INT"
        | "main"
        | "master"
        | "TEST"
        | "develop"

<semver version> ::= "v" <mayor> "." <minor> "." <patch>

<mayor> ::= <digits>

<minor> ::= <digits>

<patch> ::= <digits>

<artifact_repo> ::= <docker_hub_org> "/" <docker_hub_repo>
# TODO: add more alternatives for <artifact_repo>

# <digits> matches /\d+/
# <docker_hub_org> is any valid DockerHub organization name
# <docker_hub_repo> is any valid DockerHub repository name
# <source_code_repo_url> is any valid URL

# Full regex: /(?:(?<artifact_repo>(?<docker_hub_org>[\w-]+)\/(?<docker_hub_repo>[\w-]+)):)?(?<version_name>(?<version>v(?<mayor>\d+)\.(?<minor>\d+)\.(?<patch>\d+))|(?<name>INT|main|master|TEST|develop))[_\+](?<source_code_commit_id>\w+)(?:@(?<source_code_repo_url>.+))?/
```

### Hamekoz GitHub Packages

To use our generated artifacts, follow the next steps

- Generate a [GitHub personal access token](https://github.com/settings/tokens/new) with at least `read:packages` permission
- Set and use the environment variable `Hamekoz_GITHUB_PACKAGES_TOKEN` with the token generated

### Configure Hamekoz GitHub Packages as NuGet source

- Use or adapt the [`nuget.config`](https://github.com/hamekoz/.github/blob/main/dotnet-examples/nuget.config) file example into the required repository
