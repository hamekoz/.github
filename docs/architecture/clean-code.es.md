# Clean Code

Referencia principal: _Clean Code_ — Robert C. Martin.

Este documento define los criterios de código limpio para todos los proyectos de la organización,
en cualquier lenguaje. Los ejemplos específicos de C# reflejan las prácticas promovidas por el
proyecto de referencia `CartaUniversal`.

---

## Nombres

## # Obligatorio — Nombres

- Los nombres deben revelar la intención: `GetUserByIdAsync` en lugar de `GetU` o `Process`.
- Sin abreviaciones crípticas: `customerAccount`, no `ca` ni `custAcc`.
- Sin palabras genéricas sin significado: `Manager`, `Processor`, `Handler`, `Helper` a solas.
- Los booleanos se nombran como afirmaciones: `isActive`, `hasPermission`, `canDelete`.
- Las colecciones en plural: `orders`, `activeUsers`.

## # Obligatorio — Variables nombradas por contenido, no por tipo

**Objetivo**: el código se lee como prosa, sin necesidad de ver la asignación.

```csharp
// ❌ los nombres describen tipo/tecnología
var returnListFsharp = TzolkinDate.GetNextList(5, tzolkinDate, DateTime.UtcNow.Date);
var strVal = "user@example.com";
var tmpList = users.Where(u => u.IsActive).ToList();

// ✅ los nombres describen contenido
var kinBirthdayList = TzolkinDate.GetNextList(5, tzolkinDate, DateTime.UtcNow.Date);
var userEmail = "user@example.com";
var activeUsers = users.Where(u => u.IsActive).ToList();
```

Sufijos como `Dto`, `Temp`, `List`, `Str` solo aparecen cuando aportan comprensión genuinamente.
No codificar el detalle de implementación (framework, tecnología, tipo de colección) en el nombre.

## # Recomendado — Nombres

- Los nombres de clases son sustantivos; los de métodos son verbos.

---

## Funciones / Métodos

## # Obligatorio — Funciones

- **Una sola responsabilidad**: cada función hace una sola cosa, y bien.
- **Pequeñas**: preferiblemente menos de 20 líneas. Si necesita scroll, probablemente hace
  demasiado.
- **Un nivel de abstracción por función**: no mezclar lógica de alto nivel con detalles de
  implementación.
- **Sin efectos secundarios ocultos**: si modifica estado externo, debe ser evidente.
- **Máximo 3 parámetros**; más sugiere un objeto de parámetros o refactorizar.
- **Métodos pequeños con responsabilidades delegadas**: extraer helpers en vez de god functions.

## # Recomendado — Funciones

- Evitar parámetros booleanos que cambian el comportamiento; preferir dos funciones separadas.

## # Obligatorio — Tipos de retorno explícitos

Preferir `record`/clases de retorno sobre tuplas cuando el resultado tiene significado de dominio.

---

## Comentarios

## # Obligatorio — Comentarios

- **El código debe explicarse a sí mismo**; los comentarios son un último recurso.
- No comentar código obsoleto: eliminarlo.
- No comentar lo obvio: `i++; // incrementa i`.

## # Permitido y valioso

- Comentarios de advertencia sobre consecuencias no obvias.
- Comentarios `TODO` con contexto y responsable: `// TODO(juan): remove after migrating to v2`.
- Documentación de API pública (XML docs en .NET).
- Explicación de algoritmos complejos o decisiones de diseño no evidentes.

---

## Formato y estructura

Ver [reglas de formato por lenguaje](../code-format/README.md).

## # Obligatorio — Formato

- Consistencia en todo el archivo y el proyecto.
- El código relacionado va junto; el no relacionado, separado.
- Líneas cortas: máximo 110 caracteres.

---

## Convenciones de naming (C#)

| Elemento                       | Convención         | Ejemplo                               |
| ------------------------------ | ------------------ | ------------------------------------- |
| Clases / Métodos / Propiedades | `PascalCase`       | `OrderService`, `GetOrderAsync`       |
| Campos privados                | `_camelCase`       | `_logger`, `_repository`              |
| Constantes                     | `UPPER_SNAKE_CASE` | `MAX_STOCK_ITEMS`, `DEFAULT_TIMEZONE` |
| Métodos async                  | Sufijo `Async`     | `GetOrderAsync`                       |

## # Named arguments para intención clara

Cuando se llaman métodos con constantes, `null`, o tipos que requieren inferencia, usar
parámetros nombrados:

```csharp
// ❌ ¿qué significan estos valores?
var birthUtc = ToUtc(dateWithNoon, new TimeOnly(12, 0), DefaultTimeZoneId, null, null);

// ✅ intención cristalina
var birthUtc = ToUtc(
    date: dateWithNoon,
    timeOfBirth: new TimeOnly(hour: 12, minute: 0),
    timeZoneId: DefaultTimeZoneId,
    latitude: null,
    longitude: null);
```

---

## Manejo de errores

## # Obligatorio — Manejo de errores

- **Nunca ignorar excepciones en silencio** (`catch { }` vacío prohibido).
- Los errores deben contener contexto suficiente; loguear con `ILogger` usando logging
  estructurado antes de hacer fallback.
- Preferir excepciones sobre códigos de error de retorno.
- No usar excepciones para flujo de control normal.

```csharp
catch (InvalidOperationException ex)
{
    _logger.LogWarning(ex, "Stock item {ItemId} not found", itemId);
    throw; // o fallback explícito
}
```

## # Recomendado — Manejo de errores

- Crear tipos de excepción específicos del dominio.
- Manejar los errores en la capa más apropiada (no atrapar y relanzar sin agregar contexto).

## # Structured logging

```csharp
// ❌ concatenación de strings — no buscable
_logger.LogInformation("User " + userId + " created chart for " + date);

// ✅ parámetros estructurados
_logger.LogInformation("User {UserId} created chart for date {Date}", userId, date);
```

---

## Async/Await

- Todos los métodos async llevan el sufijo `Async`.
- Nunca `.Result` ni `.Wait()` (riesgo de deadlock) — async all the way.
- Propagar `CancellationToken` en toda la cadena async.

```csharp
// ✅ correcto
var chart = await chartService.GetChartAsync(id);
var isValid = await ValidateChartAsync(chart, cancellationToken);
```

---

## Inyección de dependencias

- Todas las dependencias de servicios se inyectan por constructor (preferir primary constructors).
- `ILogger<T>` es obligatorio en la firma de los constructores de servicios; nunca opcional/null
  en código de producción.
- Nunca instanciar `new ConcreteService()` en código de negocio.

```csharp
public sealed class StockService(IStockStore store, ILogger<StockService> logger) : IStockService
{
    // parámetros del primary constructor usados directamente; sin backing fields redundantes
}
```

---

## Anti-patterns a evitar

| Anti-Pattern                 | Problema                         | Solución                                                   |
| ---------------------------- | -------------------------------- | ---------------------------------------------------------- |
| **Magic numbers/strings**    | Valores sin contexto             | Extraer a constantes nombradas                             |
| **God Objects**              | Clases que hacen demasiado       | Dividir en responsabilidades                               |
| **Primitive Obsession**      | `string`/`int` en lugar de tipos | Value objects: `record OrderId(int Value)`                 |
| **Long Parameter Lists**     | Métodos con 5+ parámetros        | Data Transfer Object (DTO)                                 |
| **Flag Parameters**          | `bool includeDeleted`            | Métodos separados: `GetActiveAsync()`, `GetDeletedAsync()` |
| **Comments Instead of Code** | Lógica oscura + comentarios      | Refactorizar a código auto-documentado                     |

---

## Principios

## # SOLID

| Principio                     | Resumen                                                                    |
| ----------------------------- | -------------------------------------------------------------------------- |
| **S** — Single Responsibility | Una clase, una razón para cambiar                                          |
| **O** — Open/Closed           | Abierta para extensión, cerrada para modificación                          |
| **L** — Liskov Substitution   | Las subclases deben ser intercambiables por sus bases                      |
| **I** — Interface Segregation | Interfaces pequeñas y específicas; no forzar implementaciones innecesarias |
| **D** — Dependency Inversion  | Depender de abstracciones, no de implementaciones concretas                |

## # DRY, KISS, YAGNI

| Principio | Descripción                                                            |
| --------- | ---------------------------------------------------------------------- |
| **DRY**   | Cada pieza de conocimiento tiene una representación única y no ambigua |
| **KISS**  | Preferir la solución más simple que resuelva el problema               |
| **YAGNI** | No agregar funcionalidad hasta que sea necesaria                       |

---

## Code review — criterios de aceptación

- [ ] Nombres de variables/métodos/clases claros y consistentes con el dominio; las variables se
      nombran por su contenido, no por su tipo.
- [ ] Funciones pequeñas con una sola responsabilidad.
- [ ] Sin código muerto, comentado ni `catch` vacíos.
- [ ] Excepciones manejadas y logueadas con logging estructurado.
- [ ] Tests que cubren el comportamiento nuevo o modificado.
- [ ] Sin duplicación evitable.
- [ ] El código nuevo no introduce dependencias circulares.
