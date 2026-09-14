# Testing — .NET (C#)

Defines the shared testing criterion for all .NET projects of the organization. It applies to
new projects and to existing projects being migrated to the current standards.

```text
+------------------------------------------------------------------+
| Standard (required)                                  | Example    |
+------------------------------------------------------------------+
| Test framework             | xUnit (not MSTest, not NUnit)       |
| Runner (new /.NET 10)      | Microsoft Testing Platform (MTP)    |
| Runner (legacy)            | VSTest (transitional, see below)   |
| Test doubles               | Hand-written Fake*/Stub* classes    |
| Code coverage driver       | coverlet.MTP (MTP) / collector     |
+------------------------------------------------------------------+
```

---

## Runner model

### New projects (.NET 10): Microsoft Testing Platform (MTP)

MTP is the standard runner for all new test projects and for projects that target .NET 10.
Test projects are console applications (`OutputType = Exe`) executed by `dotnet test`.

- Packages: `xunit.v3` + `coverlet.MTP`.
- `global.json` declares `"test": { "runner": "Microsoft.Testing.Platform" }`.
- Coverage is collected with the `--coverlet` flag and the `--coverlet-output-format` options.

Reference implementation: `Hamekoz.NET.Sdk.Internal` tests.

### Legacy projects: VSTest (transitional)

Projects still on the VSTest runner (for example `CartaUniversal.Tests`) keep working as-is and
are considered transitional. They are not migrated as part of this criterion; migration happens
only when the project is touched for other reasons.

- Packages: `xunit` (2.9.x), `Microsoft.NET.Test.Sdk`, `xunit.runner.visualstudio`, `coverlet.collector`.
- Coverage is collected with the VSTest data collector: `--collect:"XPlat Code Coverage"`.

---

## Test project setup

### Central package management (`Directory.Packages.props`)

```xml
<PropertyGroup>
  <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
</PropertyGroup>

<ItemGroup>
  <!-- MTP runner -->
  <PackageVersion Include="coverlet.MTP" Version="10.0.1" />
  <PackageVersion Include="xunit.v3" Version="4.0.0" />
  <!-- VSTest runner (legacy only) -->
  <PackageVersion Include="Microsoft.NET.Test.Sdk" Version="18.9.0" />
  <PackageVersion Include="coverlet.collector" Version="10.0.1" />
  <PackageVersion Include="xunit" Version="2.9.3" />
  <PackageVersion Include="xunit.runner.visualstudio" Version="4.0.0" />
</ItemGroup>
```

### `global.json`

```json
{
  "sdk": {
    "version": "10.0.111",
    "rollForward": "latestFeature"
  },
  "test": {
    "runner": "Microsoft.Testing.Platform"
  }
}
```

### Test project `<ProjectName>.csproj` (MTP)

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
    <UseMicrosoftTestingPlatformRunner>true</UseMicrosoftTestingPlatformRunner>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="coverlet.MTP" />
    <PackageReference Include="xunit.v3" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\..\src\MyPackage\MyPackage.csproj" />
  </ItemGroup>

  <ItemGroup>
    <Using Include="Xunit" />
  </ItemGroup>

</Project>
```

`UseMicrosoftTestingPlatformRunner=true` registers the `coverlet.MTP` extension. Without it the
`--coverlet` option is not recognized by the test application.

### Test project `<ProjectName>.csproj` (VSTest, legacy)

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <IsPackable>false</IsPackable>
    <IsTestProject>true</IsTestProject>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" />
    <PackageReference Include="coverlet.collector" />
    <PackageReference Include="xunit" />
    <PackageReference Include="xunit.runner.visualstudio" />
  </ItemGroup>

  <ItemGroup>
    <Using Include="Xunit" />
  </ItemGroup>

</Project>
```

---

## Commands

### Local — MTP runner

```sh
# All tests
dotnet test --project tests/MyPackage.Tests

# Tests with coverage (cobertura + json)
dotnet test --project tests/MyPackage.Tests \
  --coverlet \
  --coverlet-output-format cobertura \
  --coverlet-output-format json
```

Notes:

- `--coverlet-output-format` must be repeated once per format. A comma-separated value such as
  `"cobertura,json"` is **not** accepted on the command line.
- Coverlet writes to the `TestResults` directory of the test project output
  (`bin/<Config>/net10.0/TestResults/`): `coverage.cobertura.<timestamp>.xml` and `coverage.<timestamp>.json`.

### Local — VSTest runner

```sh
dotnet test --collect:"XPlat Code Coverage"
```

### CI — shared workflow (`hamekoz/.github/.github/workflows/dotnet.yml`)

The workflow runs `dotnet test --no-build --configuration Release ${{ inputs.test_arguments }}`.

- The default `test_arguments` value is `--collect:"XPlat Code Coverage"` and is kept for VSTest
  compatibility. Legacy projects do not need to configure anything.
- MTP projects must override `test_arguments`:

```yaml
jobs:
  dotnet:
    uses: hamekoz/.github/.github/workflows/dotnet.yml@main
    secrets: inherit
    with:
      test_arguments: >-
        --coverlet
        --coverlet-output-format cobertura
        --coverlet-output-format json
```

The workflow uploads the coverage report to Codecov after the test step. Codecov auto-detects
`coverage.cobertura*.xml` under `TestResults`; no additional input is required.

---

## Test conventions

### File layout

- `tests/<ProjectName>.Tests/` mirrors the source layout.
- Test files are placed flat in the test project root: one file `XServiceTests.cs` per tested
  class.
- Hand-written doubles live next to the tests: `FakeRepository.cs`, `FakeEmailSender.cs`.

### Naming

- Test classes: `<ClassName>Tests` (e.g. `StockServiceTests`).
- Test methods: `Method_Condition_Outcome` (e.g. `GetById_WhenIdDoesNotExist_ReturnsNull`) or
  `Should_<Outcome>_When_<Condition>`.
- Names describe the scenario in plain English, no `Test1`, `Test2`.

### Structure

- Follow **Arrange / Act / Assert** (AAA) with a blank line between sections.
- One concept per test (`[Fact]` for a single path, `[Theory]` + `[InlineData]` for data-driven).
- Tests are **F.I.R.S.T.**: fast, independent, repeatable, self-validating, timely.

### Test doubles

- Prefer hand-written `Fake*`/`Stub*` classes over mocking frameworks. They keep tests explicit
  and are trivial for repository/service seams.
- Mocking frameworks (e.g. Moq) are allowed only when a fake would require copying large amounts
  of production behavior; justify it in the PR.
- Doubles must implement the same interface and never embed test data magic.

### Determinism

- Never depend on `DateTime.Now`, environment timezones, random values, or shared static state.
- Inject `TimeProvider` (or a clock interface) and use `ITestOutputHelper` for diagnostics.
- Each test creates its own instances; do not reuse mutable fixtures across tests.

### Coverage thresholds

Coverage is a quality gate, not an absolute objective. Suggested minimums per module:

| Metric          | Minimum |
| --------------- | ------- |
| Line coverage   | >= 80%  |
| Branch coverage | >= 70%  |
| CRAP score      | <= 30   |

- Track critical paths: validation, invariants, error handling, security decisions.
- Do not chase 100%. Prefer meaningful assertions over line-count inflation.

---

## Definition of done (checklist)

- [ ] New behavior has at least one failing test before the implementation (or a regression test).
- [ ] Tests are deterministic and isolated.
- [ ] Doubles are hand-written `Fake*`/`Stub*`; mocking frameworks used only with justification.
- [ ] Coverage runs in CI for the project (MTP: `--coverlet`; VSTest: default collector).
- [ ] `dotnet format --verify-no-changes`, `dotnet build`, and `dotnet test` all pass.
