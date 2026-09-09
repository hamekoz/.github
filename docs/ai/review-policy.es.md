# Política de Revisión Humana — Cambios generados por IA

Todo código generado o modificado por un agente de IA **requiere revisión y aprobación humana**
antes de ser mergeado a ramas protegidas.

---

## Principio rector

> Los agentes IA son asistentes, no decision-makers. Un humano es siempre responsable del código
> que entra al repositorio.

---

## Proceso obligatorio

```text
Tarea solicitada al agente
        │
        ▼
Agente genera cambios en una rama de trabajo
con prefijo semántico (feature/, fix/, docs/, ...)
        │
        ▼
PR abierto por el agente o por el solicitante
        │
        ▼
CI ejecuta todos los checks automáticos
        │
        ▼
Revisión humana obligatoria ──► Observaciones ──► Agente itera
        │
        ▼ (aprobado)
Merge a rama base
        │
        ▼
Registrar la tarea en el historial de tareas IA del repositorio
```

---

## Criterios de revisión por categoría

### Cambios de lógica de negocio

- [ ] ¿La lógica implementada es correcta con respecto al requerimiento?
- [ ] ¿Se manejan los edge cases relevantes?
- [ ] ¿Hay tests que cubran el comportamiento nuevo?
- [ ] ¿La implementación es coherente con el diseño existente?

### Cambios de arquitectura

- [ ] ¿Se respetan las capas de Clean Architecture?
- [ ] ¿Las dependencias van en la dirección correcta?
- [ ] ¿No se introdujeron dependencias circulares?

### Cambios en CI/CD o configuración

- [ ] ¿Se entiende el efecto de cada cambio en el workflow?
- [ ] ¿No se redujo el nivel de verificación del pipeline?
- [ ] ¿No se exponen secrets o variables sensibles?

### Cambios en dependencias

- [ ] ¿La nueva dependencia está justificada?
- [ ] ¿Se verificó que no tenga CVEs conocidas?
- [ ] ¿La versión está fijada apropiadamente?

### Seguridad (obligatorio para cualquier cambio)

- [ ] ¿No hay secretos, tokens ni credenciales hardcodeadas?
- [ ] ¿Se validan todos los inputs externos?
- [ ] ¿No se introdujeron vulnerabilidades conocidas?

---

## Aprobaciones requeridas

| Tipo de cambio                               | Aprobaciones mínimas        |
| -------------------------------------------- | --------------------------- |
| Documentación / comentarios                  | 1                           |
| Tests, CI                                    | 1                           |
| Código de aplicación                         | 1                           |
| Cambios de arquitectura o diseño             | 2                           |
| Cambios en seguridad o autenticación         | 2 (uno debe ser maintainer) |
| Cambios en workflows compartidos (`.github`) | 2 maintainers               |

---

## Qué debe agregar el revisor al aprobar

En el comentario de aprobación o en la descripción del merge, el revisor debe confirmar:

- Que revisó el diff completo.
- Que los criterios aplicables de la lista anterior fueron verificados.
- Si hay deuda técnica o seguimiento necesario, crear un issue vinculado.

---

## Cuándo rechazar un PR generado por IA

- El código no sigue las convenciones de la organización.
- Los tests no pasan o no existen para el comportamiento nuevo.
- El alcance del cambio excede lo solicitado (el agente modificó más de lo pedido).
- Hay dudas razonables sobre seguridad que no fueron resueltas.
- La lógica es correcta pero el diseño no es mantenible.

---

## Registro de la revisión

Después de cada merge de cambios generados por IA, actualizar el
[historial de tareas IA](./ai-task-log-template.md) del repositorio con el resultado de la
revisión.