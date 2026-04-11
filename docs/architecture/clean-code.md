# Clean Code

Referencia principal: _Clean Code_ — Robert C. Martin.

Este documento define los criterios de código limpio aplicables a todos los proyectos de la organización, independientemente del lenguaje.

---

## Nombres

### Obligatorio — Nombres

- Los nombres deben revelar la intención: `getUserById` en lugar de `getU` o `process`.
- Evitar abreviaciones crípticas: `customerAccount` no `ca` o `custAcc`.
- Evitar palabras genéricas sin significado: `Manager`, `Processor`, `Handler`, `Helper` a solas.
- Los booleanos se nombran como afirmaciones: `isActive`, `hasPermission`, `canDelete`.
- Las colecciones en plural: `orders`, `userList`.

### Recomendado — Nombres

- Los nombres de clases son sustantivos; los de métodos son verbos.

---

## Funciones / Métodos

### Obligatorio — Funciones

- **Una sola responsabilidad**: cada función hace una sola cosa y la hace bien.
- **Pequeñas**: preferiblemente menos de 20 líneas. Si necesita scroll, probablemente hace demasiado.
- **Un nivel de abstracción por función**: no mezclar lógica de alto nivel con detalles de implementación en la misma función.
- **Sin efectos secundarios ocultos**: si una función modifica estado externo, debe ser evidente en su nombre o documentación.
- **Máximo 3 parámetros**; si necesita más, considerar un objeto de parámetros o refactorizar.

### Recomendado — Funciones

- Evitar parámetros booleanos que cambian el comportamiento; preferir dos funciones separadas.

---

## Comentarios

### Obligatorio — Comentarios

- **El código debe explicarse a sí mismo**; los comentarios son un último recurso.
- No comentar código obsoleto: eliminarlo. El control de versiones guarda el historial.
- No comentar lo obvio: `i++; // incrementa i`.

### Permitido y valioso

- Comentarios de advertencia sobre consecuencias no obvias.
- Comentarios `TODO` con contexto y responsable: `// TODO(juan): remove after migrating to v2`.
- Documentación de API pública (XML docs en .NET, RDoc en Ruby).
- Explicación de algoritmos complejos o decisiones de diseño no evidentes.

---

## Formato y estructura

Ver [reglas de formato por lenguaje](../code-format/README.md).

### Obligatorio — Formato

- Consistencia en todo el archivo y el proyecto.
- El código relacionado va junto; el no relacionado, separado.
- Líneas cortas: máximo 110–120 caracteres.

---

## Manejo de errores

### Obligatorio — Manejo de errores

- **Nunca ignorar excepciones en silencio** (`catch { }` vacío).
- Los errores deben contener contexto suficiente para diagnosticar el problema.
- Preferir excepciones sobre códigos de error de retorno.
- No usar excepciones para flujo de control normal.

### Recomendado — Manejo de errores

- Crear tipos de excepción específicos del dominio.
- Manejar los errores en el nivel más apropiado de la aplicación (no atrapar y relanzar sin agregar contexto).

---

## Tests

### Obligatorio — Tests

- **F.I.R.S.T.**: Fast, Independent, Repeatable, Self-validating, Timely.
- Un concepto por test.
- Nombres descriptivos: `Should_ReturnError_When_EmailIsInvalid`.
- Los tests no deben depender de orden de ejecución ni de estado compartido mutable.

### Recomendado — Tests

- Cobertura de código como guía, no como objetivo absoluto. Priorizar calidad sobre porcentaje.
- Seguir el patrón Arrange / Act / Assert.

---

## Principios SOLID (resumen)

| Principio                     | Descripción resumida                                                       |
| ----------------------------- | -------------------------------------------------------------------------- |
| **S** — Single Responsibility | Una clase, una razón para cambiar                                          |
| **O** — Open/Closed           | Abierta para extensión, cerrada para modificación                          |
| **L** — Liskov Substitution   | Las subclases deben ser intercambiables por sus bases                      |
| **I** — Interface Segregation | Interfaces pequeñas y específicas; no forzar implementaciones innecesarias |
| **D** — Dependency Inversion  | Depender de abstracciones, no de implementaciones concretas                |

---

## DRY, KISS, YAGNI

| Principio                           | Descripción                                                                 |
| ----------------------------------- | --------------------------------------------------------------------------- |
| **DRY** (Don't Repeat Yourself)     | Cada pieza de conocimiento debe tener una representación única y no ambigua |
| **KISS** (Keep It Simple, Stupid)   | Preferir la solución más simple que resuelva el problema                    |
| **YAGNI** (You Ain't Gonna Need It) | No agregar funcionalidad hasta que sea necesaria                            |

---

## Code review — criterios de aceptación

Al revisar código, verificar:

- [ ] El nombre de variables, métodos y clases es claro y consistente con el dominio.
- [ ] Las funciones son pequeñas y tienen una sola responsabilidad.
- [ ] No hay comentarios de código obsoleto ni código comentado.
- [ ] Las excepciones se manejan correctamente.
- [ ] Hay tests que cubren el comportamiento nuevo o modificado.
- [ ] No hay duplicación evitable.
- [ ] El código nuevo no introduce dependencias circulares.
