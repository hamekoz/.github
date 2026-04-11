# CI/CD — Ruby on Rails

Guía de CI/CD para proyectos Ruby on Rails de la organización.

Ver la [guía general de CI/CD](./README.md) para el pipeline mínimo y los principios.

---

## Stack de herramientas

| Herramienta                                               | Propósito                             |
| --------------------------------------------------------- | ------------------------------------- |
| [RuboCop](https://rubocop.org/)                           | Lint y formato de código              |
| [RSpec](https://rspec.info/)                              | Tests unitarios e integración         |
| [SimpleCov](https://github.com/simplecov-ruby/simplecov)  | Cobertura de código                   |
| [Brakeman](https://brakemanscanner.org/)                  | Análisis de seguridad estático        |
| [bundler-audit](https://github.com/rubysec/bundler-audit) | Auditoría de vulnerabilidades en gems |
| [Codecov](https://codecov.io/)                            | Reporte de cobertura                  |

---

## Workflow de referencia para GitHub Actions

```yaml
# .github/workflows/ci.yml
name: CI/CD

on:
  push:
    branches: [main, develop]
    tags:
      - "v*.*.*"
  pull_request:
    branches: [main, develop]

jobs:
  conventions:
    if: github.event_name == 'pull_request'
    name: Conventions
    uses: hamekoz/.github/.github/workflows/conventions.yml@main

  lint:
    name: Lint (RuboCop)
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: .ruby-version
          bundler-cache: true
      - name: Run RuboCop
        run: bundle exec rubocop --no-color --format progress

  security:
    name: Security Scan
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: .ruby-version
          bundler-cache: true
      - name: Brakeman
        run: bundle exec brakeman --no-pager --exit-on-warn
      - name: bundler-audit
        run: |
          bundle exec bundle-audit check --update

  test:
    name: Tests (RSpec)
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: myapp_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    env:
      RAILS_ENV: test
      DATABASE_URL: postgres://postgres:postgres@localhost:5432/myapp_test
    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: .ruby-version
          bundler-cache: true
      - name: Setup database
        run: |
          bundle exec rails db:create db:schema:load
      - name: Run tests
        run: bundle exec rspec --format progress
      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}

  docker:
    name: Publish Docker image
    needs: [lint, test, security]
    if: github.event_name == 'push'
    uses: hamekoz/.github/.github/workflows/continuous-delivery-dockerfile.yml@main
    secrets: inherit
    with:
      docker-image-name: "my-rails-app"
```

---

## Requisitos del repositorio Rails

- `.ruby-version` en la raíz con la versión exacta de Ruby.
- `Gemfile.lock` comprometido al repositorio.
- `config/database.yml` configurable por variables de entorno (no hardcodear credenciales).
- `db/schema.rb` (no `structure.sql` salvo que sea necesario por uso de extensiones DB avanzadas).

---

## Cobertura de código con SimpleCov

```ruby
# spec/spec_helper.rb
require "simplecov"

SimpleCov.start "rails" do
  add_filter "/spec/"
  add_filter "/config/"
  minimum_coverage 80
end
```

---

## Seguridad

### Brakeman

Ejecutar en cada PR. Configurar en `.brakeman.ignore` los falsos positivos documentados.

```sh
bundle exec brakeman --no-pager
```

### bundler-audit

Verifica que no haya gems con CVEs conocidas.

```sh
bundle exec bundle-audit check --update
```

---

## Versionado semántico para APIs Rails

Usar [semantic_versioning](https://github.com/jlindsey/semantic_versioning) o mantener la versión en `config/version.rb`:

```ruby
# config/version.rb
module MyApp
  VERSION = ENV.fetch("SOURCE_VERSION", "dev+unknown")
end
```

Exponer en el endpoint de salud:

```ruby
# config/routes.rb
get "/version", to: proc { [200, {}, [MyApp::VERSION]] }
```

---

## Estrategia de branches y deploys

Seguir la [estrategia de branching](../conventions/branching.md) organizacional.

Las imágenes Docker se publican en GHCR con tags según Semantic Versioning:

```text
ghcr.io/hamekoz/my-rails-app:v1.2.3
ghcr.io/hamekoz/my-rails-app:v1.2
ghcr.io/hamekoz/my-rails-app:develop
```
