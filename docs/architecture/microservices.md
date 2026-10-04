# Principios de Microservicios

Este documento define las reglas y heurísticas para el diseño y operación de microservicios en la organización.

---

## Definición de límites de contexto (Bounded Context)

- Cada microservicio encapsula **un único contexto de negocio** bien definido.
- El servicio es dueño de sus datos: ningún otro servicio accede directamente a su base de datos.
- Usar el **lenguaje ubicuo** del dominio en la nomenclatura del servicio, sus APIs y su código.
- Señales de que un servicio está mal dimensionado:
  - Necesita leer datos de otro servicio para completar la mayoría de sus operaciones.
  - Dos equipos distintos modifican frecuentemente el mismo servicio.
  - El servicio tiene varias razones de escalado diferentes.

---

## Contratos de API

## # Reglas obligatorias

- Las APIs son **versionadas desde el primer release**: `/v1/resource`.
- Los cambios en la API siguen Semantic Versioning: breaking changes → nueva versión mayor.
- Las APIs se documentan con **OpenAPI / Swagger** y el spec se versiona en el repositorio.
- Las respuestas de error siguen un formato estándar consistente:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "Validation failed",
  "status": 400,
  "errors": {
    "email": ["Email format is invalid"]
  }
}
```

## # Compatibilidad hacia atrás

- Nunca eliminar ni renombrar campos de respuesta existentes en la misma versión.
- Los campos nuevos en respuestas son aditivos y no rompen clientes existentes.
- Los campos opcionales nuevos en request no rompen compatibilidad.
- Mantener versiones antiguas activas durante un período de deprecación documentado.

---

## Comunicación entre servicios

## # Síncrona (HTTP/gRPC)

- Usar para operaciones que requieren respuesta inmediata.
- Implementar **circuit breaker** y **retry con backoff exponencial**.
- Documentar explícitamente las dependencias síncronas (riesgo de cascada de fallos).

## # Asíncrona (mensajería / eventos)

- Preferir comunicación por eventos para operaciones que no requieren respuesta inmediata.
- Los eventos deben ser inmutables y tener un contrato de schema versionado.
- Usar nombres de eventos en pasado: `OrderPlaced`, `PaymentProcessed`, `UserRegistered`.

---

## Ownership y responsabilidades

| Aspecto           | Regla                                                               |
| ----------------- | ------------------------------------------------------------------- |
| **Repositorio**   | Un servicio, un repositorio.                                        |
| **Base de datos** | Un servicio, una base de datos (o schema exclusivo).                |
| **Equipo**        | Cada servicio tiene un equipo/dueño identificado en el `README.md`. |
| **On-call**       | El equipo dueño es responsable de la operación del servicio.        |

---

## Observabilidad

## # Obligatorio

- **Logs estructurados** (JSON) con los campos mínimos:
  - `timestamp`, `level`, `service`, `traceId`, `message`
- **Métricas** expuestas en `/metrics` (Prometheus) o equivalente:
  - Latencia p50/p95/p99, tasa de error, throughput.
- **Health check** en `/health` o `/healthz`:
  - Liveness: ¿está el proceso vivo?
  - Readiness: ¿puede atender tráfico? (valida conexión a BD y deps críticos)
- **Trace ID propagado** en todos los requests inter-servicio (header `X-Trace-Id` o `traceparent` W3C).

## # Recomendado

- Dashboard centralizado con indicadores clave por servicio.
- Alertas automáticas en base a métricas (tasa de error > umbral, latencia elevada).

---

## Resiliencia

- **Idempotencia**: las operaciones de escritura deben ser seguras de reintentar.
- **Timeouts explícitos**: nunca esperar indefinidamente a un servicio externo.
- **Graceful degradation**: si un servicio no crítico falla, el sistema debe seguir operando de forma reducida.
- **Bulkhead**: aislar pools de recursos para que el fallo de un servicio no agote los recursos de otro.

---

## Seguridad entre servicios

- Autenticación inter-servicio mediante tokens JWT firmados o mTLS.
- Cada servicio valida los tokens de los requests entrantes.
- Los servicios internos no están expuestos directamente a internet; usan API Gateway o ingress.
- Principio de mínimo privilegio: cada servicio solo tiene acceso a los datos y servicios que necesita.

---

## Criterios de aceptación en code review

- [ ] El servicio tiene dueño documentado en `README.md`.
- [ ] La API está versionada y documentada con OpenAPI.
- [ ] El servicio expone `/health` con liveness y readiness.
- [ ] Los logs son estructurados e incluyen `traceId`.
- [ ] Ningún otro servicio accede directamente a la base de datos de este servicio.
- [ ] Los cambios breaking en la API crean una nueva versión, no modifican la actual.
- [ ] Se implementa circuit breaker o retry para llamadas a servicios externos.
