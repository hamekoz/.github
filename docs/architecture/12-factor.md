# 12 Factor App — Checklist por tipo de servicio

Referencia: [12factor.net](https://12factor.net/)

Aplicar este checklist durante el diseño y revisión de cualquier servicio. Los ítems marcados como **obligatorio** son requisitos mínimos para todos los proyectos.

---

## Factor 1 — Codebase

> Un codebase, múltiples deploys.

- [x] **Obligatorio**: El código vive en un único repositorio Git.
- [x] **Obligatorio**: No compartir código entre apps copiando archivos; usar paquetes o librerías versionadas.
- [ ] Recomendado: Un repositorio por servicio/aplicación independiente.

---

## Factor 2 — Dependencies

> Declarar y aislar dependencias explícitamente.

- [x] **Obligatorio**: Todas las dependencias declaradas en un manifiesto (`*.csproj`, `Gemfile`, `package.json`).
- [x] **Obligatorio**: Nunca asumir dependencias del sistema operativo instaladas globalmente.
- [x] **Obligatorio**: Lockfile comprometido al repositorio (`packages.lock.json`, `Gemfile.lock`).

---

## Factor 3 — Config

> Guardar la configuración en el entorno, no en el código.

- [x] **Obligatorio**: Ningún secreto, URL de base de datos ni credencial comprometida en el código fuente.
- [x] **Obligatorio**: Variables de entorno para toda configuración que varía entre ambientes.
- [x] **Obligatorio**: Usar GitHub Secrets para secrets en CI/CD.
- [ ] Recomendado: Centralizar la configuración de secrets con una herramienta dedicada (ej. Azure Key Vault, AWS Secrets Manager).

---

## Factor 4 — Backing Services

> Tratar los servicios de respaldo como recursos adjuntos.

- [x] **Obligatorio**: Bases de datos, colas, SMTP, etc. configurados por URL/variable de entorno.
- [x] **Obligatorio**: El código no distingue entre servicios locales y externos.
- [ ] Recomendado: Poder intercambiar un backing service sin cambios de código, solo de configuración.

---

## Factor 5 — Build, Release, Run

> Separar estrictamente las etapas de build, release y run.

- [x] **Obligatorio**: El pipeline CI/CD distingue: build → test → package → release → deploy.
- [x] **Obligatorio**: Los artefactos de build son inmutables; no modificar en release o run.
- [x] **Obligatorio**: Cada release tiene un identificador único (versión semántica + SHA de commit).

---

## Factor 6 — Processes

> Ejecutar la app como uno o más procesos sin estado.

- [x] **Obligatorio**: Los procesos de la aplicación son stateless.
- [x] **Obligatorio**: No almacenar estado de sesión en memoria o disco local; usar caché externa o base de datos.
- [ ] API: Sin sesiones de servidor; usar tokens JWT.
- [ ] Worker: Idempotente — puede ejecutarse múltiples veces con el mismo input sin efectos colaterales.

---

## Factor 7 — Port Binding

> Exportar servicios via port binding.

- [x] **Obligatorio**: La app es autosuficiente y expone su servicio por un puerto configurado.
- [ ] Recomendado: Usar variables de entorno para definir el puerto (`PORT`, `ASPNETCORE_URLS`).

---

## Factor 8 — Concurrency

> Escalar mediante el modelo de procesos.

- [ ] Recomendado: Diseñar para escalar horizontalmente agregando instancias.
- [ ] Recomendado: Separar preocupaciones en tipos de proceso distintos (web, worker, scheduler).

---

## Factor 9 — Disposability

> Maximizar robustez con arranque rápido y apagado graceful.

- [ ] Recomendado: Tiempo de arranque menor a 30 segundos.
- [x] **Obligatorio para APIs**: Manejar señales `SIGTERM` para terminar conexiones activas antes de apagar.
- [ ] Recomendado para Workers: Retornar tareas incompletas a la cola si el proceso termina inesperadamente.

---

## Factor 10 — Dev/Prod Parity

> Mantener desarrollo, staging y producción lo más similares posible.

- [x] **Obligatorio**: Usar el mismo motor de base de datos en desarrollo y producción.
- [x] **Obligatorio**: CI/CD en todos los ambientes desde el inicio del proyecto.
- [ ] Recomendado: Usar Docker para garantizar paridad de entorno.

---

## Factor 11 — Logs

> Tratar los logs como flujos de eventos.

- [x] **Obligatorio**: Los logs van a stdout/stderr; no escribir a archivos locales.
- [x] **Obligatorio**: Incluir contexto en cada log: request ID, usuario, operación.
- [ ] Recomendado: Usar formato estructurado (JSON) para facilitar ingesta en sistemas de observabilidad.
- [ ] Recomendado: Niveles de log configurables por variable de entorno (`LOG_LEVEL`).

---

## Factor 12 — Admin Processes

> Ejecutar tareas administrativas como procesos one-off.

- [x] **Obligatorio**: Las migraciones de base de datos son procesos separados del arranque de la app.
- [ ] Recomendado: Las tareas de mantenimiento (seed, clean, export) se ejecutan como jobs/scripts versionados.

---

## Checklist rápido por tipo de proyecto

## # API / Microservicio

Factores críticos: 3, 5, 6, 8, 9, 11.

## # Worker / Background Job

Factores críticos: 3, 6, 9, 12.

## # Librería / NuGet Package

Factores críticos: 2, 5.

## # Frontend / SPA

Factores críticos: 3, 5, 7.
