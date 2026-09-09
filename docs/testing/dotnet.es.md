# Testing — .NET (C#)

Define el criterio de testing compartido para todos los proyectos .NET de la organización. Aplica
a proyectos nuevos y a proyectos existentes que se estén migrando a los estándares actuales.

```text
+------------------------------------------------------------------+
| Estándar (obligatorio)                               | Ejemplo    |
+------------------------------------------------------------------+
| Framework de tests           | xUnit (no MSTest, no NUnit)       |
| Runner (nuevo /.NET 10)      | Microsoft Testing Platform (MTP)  |
| Runner (legacy)              | VSTest (transitorio, ver abajo)   |
| Test doubles                 | Clases Fake*/Stub* escritas a mano|
| Driver de cobertura          | coverlet.MTP (MTP) / collector    |
+------------------------------------------------------------------+
```

---

## Modelo de runner

### Proyectos nuevos (.NET 10): Microsoft Testing Platform (MTP)

MTP es el runner estándar para todos los proyectos de test nuevos y para los que apuntan a
.NET 10. Los proyectos de test son aplicaciones de consola (`OutputType = Exe`) ejecutadas por
`dotnet test`.

- Paquetes: `xunit.v3` + `coverlet.MTP`.
- `global.json` declara `"test": { "runner": "Microsoft.Testing.Platform" }`.
- La cobertura se recolecta con el flag `--coverlet` y las opciones `--coverlet-output-format`.

Implementación de referencia: los tests de `Hamekoz.NET.Sdk.Internal`.

### Proyectos legacy: VSTest (transitorio)

Los proyectos que siguen en el runner VSTest (por ejemplo `CartaUniversal.Tests`) continúan
funcionando tal cual y se consideran transitorios. No se migran como parte de este criterio; la
migración ocurre solo cuando el proyecto se toca por otros motivos.

- Paquetes: `xunit` (2.9.x), `Microsoft.NET.Test.Sdk`, `xunit.runner.visualstudio`, `coverlet.collector`.
- La cobertura se recolecta con el data collector de VSTest: `--collect:"XPlat Code Coverage"`.

---

## Setup del proyecto de test

### Central Package Management (`Directory.Packages.props`)

```xml
<PropertyGroup>
  <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
</PropertyGroup>

<ItemGroup>
  <!-- Runner MTP -->
  <PackageVersion Include="coverlet.MTP" Version="10.0.1" />
  <PackageVersion Include="xunit.v3" Version="4.0.0" />
  <!-- Runner VSTest (solo legacy) -->
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

### `<ProjectName>.csproj` del proyecto de test (MTP)

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

`UseMicrosoftTestingPlatformRunner=true` registra la extensión de `coverlet.MTP`. Sin él, la
aplicación de test no reconoce la opción `--coverlet`.

### `<ProjectName>.csproj` del proyecto de test (VSTest, legacy)

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

## Comandos

### Local — runner MTP

```sh
# Todos los tests
dotnet test --project tests/MyPackage.Tests

# Tests con cobertura (cobertura + json)
dotnet test --project tests/MyPackage.Tests \
  --coverlet \
  --coverlet-output-format cobertura \
  --coverlet-output-format json
```

Notas:

- `--coverlet-output-format` debe repetirse una vez por formato. Un valor separado por comas
  como `"cobertura,json"` **no** es aceptado en la línea de comandos.
- Coverlet escribe en el directorio `TestResults` del output del proyecto de test
  (`bin/<Config>/net10.0/TestResults/`): `coverage.cobertura.<timestamp>.xml` y `coverage.<timestamp>.json`.

### Local — runner VSTest

```sh
dotnet test --collect:"XPlat Code Coverage"
```

### CI — workflow compartido (`hamekoz/.github/.github/workflows/dotnet.yml`)

El workflow ejecuta `dotnet test --no-build --configuration Release ${{ inputs.test_arguments }}`.

- El valor por defecto de `test_arguments` es `--collect:"XPlat Code Coverage"` y se mantiene por
  compatibilidad con VSTest. Los proyectos legacy no necesitan configurar nada.
- Los proyectos MTP deben sobreescribir `test_arguments`:

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

El workflow sube el reporte de cobertura a Codecov después del paso de test. Codecov auto-detecta
`coverage.cobertura*.xml` bajo `TestResults`; no se requiere configurar inputs adicionales.

---

## Convenciones de test

### Disposición de archivos

- `tests/<ProjectName>.Tests/` espeja la estructura de la fuente.
- Los archivos de test van planos en la raíz del proyecto de test: un archivo `XServiceTests.cs`
  por clase probada.
- Los doubles escritos a mano viven junto a los tests: `FakeRepository.cs`, `FakeEmailSender.cs`.

### Naming

- Clases de test: `<ClassName>Tests` (p.ej. `StockServiceTests`).
- Métodos de test: `Method_Condition_Outcome` (p.ej. `GetById_WhenIdDoesNotExist_ReturnsNull`) o
  `Should_<Outcome>_When_<Condition>`.
- Los nombres describen el escenario en inglés simple; nada de `Test1`, `Test2`.

### Estructura

- Seguir **Arrange / Act / Assert** (AAA) con una línea en blanco entre secciones.
- Un concepto por test (`[Fact]` para un camino simple, `[Theory]` + `[InlineData]` para
  data-driven).
- Los tests son **F.I.R.S.T.**: fast, independent, repeatable, self-validating, timely.

### Test doubles

- Preferir clases `Fake*`/`Stub*` escritas a mano sobre frameworks de mocking. Mantienen los tests
  explícitos y son triviales para los seams de repositorios/servicios.
- Los frameworks de mocking (p.ej. Moq) solo se permiten cuando un fake requeriría copiar grandes
  porciones de comportamiento de producción; justificarlo en el PR.
- Los doubles deben implementar la misma interfaz y nunca embeder datos mágicos de test.

### Determinismo

- Nunca depender de `DateTime.Now`, timezones del entorno, valores aleatorios o estado estático
  compartido.
- Inyectar `TimeProvider` (o una interfaz de reloj) y usar `ITestOutputHelper` para diagnóstico.
- Cada test crea sus propias instancias; no reutilizar fixtures mutables entre tests.

### Umbrales de cobertura

La cobertura es una compuerta de calidad, no un objetivo absoluto. Mínimos sugeridos por módulo:

| Métrica            | Mínimo |
| ------------------ | ------ |
| Cobertura de línea | >= 80% |
| Cobertura de ramas | >= 70% |
| CRAP score         | <= 30  |

- Medir los caminos críticos: validación, invariantes, manejo de errores, decisiones de seguridad.
- No perseguir 100%. Preferir aserciones con significado antes que inflar el conteo de líneas.

---

## Definition of done (checklist)

- [ ] El comportamiento nuevo tiene al menos un test que falla antes de la implementación (o un
      test de regresión).
- [ ] Los tests son deterministas y aislados.
- [ ] Los doubles son `Fake*`/`Stub*` escritos a mano; frameworks de mocking solo con justificación.
- [ ] La cobertura corre en CI para el proyecto (MTP: `--coverlet`; VSTest: collector por defecto).
- [ ] `dotnet format --verify-no-changes`, `dotnet build` y `dotnet test` pasan.
