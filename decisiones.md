# Decisiones técnicas — clever-indem

---
## 2026-05-05 — Back va primero (order de ejecución back → front)

- **Contexto:** Luis preguntó quién va primero cuando se implementan módulos paralelos.
- **Decisión:** Back siempre va primero. El front no puede construir sus interfaces TypeScript ni sus servicios sin conocer exactamente los DTOs y rutas que devuelve el back — incluso en modo mock.
- **Impacto:** ambos
- **Alternativas descartadas:** Paralelo puro — descartado porque el front estaría adivinando contratos y habría que re-trabajar cuando se confirmaran los DTOs reales.

---
## 2026-05-05 — Mocks en controller sin capa de servicio (deuda técnica aceptada)

- **Contexto:** Los endpoints nuevos de `analisis/` necesitaban estar listos rápido para que el front avanzara. Crear la arquitectura completa (domain + application + adapter) toma el triple de tiempo.
- **Decisión:** Los controllers de `analisis/` retornan respuestas hardcodeadas directamente, sin pasar por capa `application/` ni `domain/`. Se acepta como deuda técnica explícita hasta que se conecte MongoDB.
- **Impacto:** back
- **Alternativas descartadas:** Arquitectura hexagonal completa desde el inicio — descartado porque el back aún no tiene entidad MongoDB para `Caso` ni para `Analisis`. Construir los puertos sin la base de datos conectada sería arquitectura especulativa.

---
## 2026-05-05 — DecisionAnalisisRequest usa camelCase + @JsonProperty en vez de snake_case nativo

- **Contexto:** La convención del back (CasoRequest, RadicacionRequest) es usar campos snake_case nativos en los Request DTOs. Al crear `DecisionAnalisisRequest`, el agente usó camelCase + `@JsonProperty`, que es el patrón de los Response DTOs.
- **Decisión:** Se mantiene así por ahora para no bloquear al front. Se registra como deuda técnica 🟡 a corregir antes de la integración real con MongoDB.
- **Impacto:** back
- **Alternativas descartadas:** Corregir inmediatamente — descartado porque implicaba re-generar la parte del ticket-front y re-ejecutar el agente front, con riesgo de introducir nuevos bugs.

---
## 2026-05-05 — effect() en constructor en vez de ngOnChanges para signal inputs

- **Contexto:** Angular 20 usa signal inputs (`input()`). Los signal inputs **no disparan `ngOnChanges`**. El agente front usó `ngOnChanges` por defecto, lo que dejó `ModalAnalisisComponent` siempre vacío.
- **Decisión:** Todos los componentes que necesiten reaccionar a cambios en signal inputs deben usar `effect()` en el constructor. `ngOnChanges` queda prohibido para signal inputs en este proyecto.
- **Impacto:** front
- **Alternativas descartadas:** `ngOnChanges` — incompatible con signal inputs en Angular 20. `toObservable()` — más verboso y agrega dependencia de RxJS donde no es necesario.

---
## 2026-05-05 — claude -p via stdin (no --directory flag)

- **Contexto:** Al intentar usar `claude -p --directory /ruta/repo`, el CLI rechazó el flag con error.
- **Decisión:** Para ejecutar agentes no interactivos en un directorio específico, se usa `cd /ruta/repo && claude -p < /tmp/prompt.txt`. El prompt se escribe primero a un archivo temporal.
- **Impacto:** proceso de coordinación (no afecta código)
- **Alternativas descartadas:** `--directory` flag — no existe en esta versión del CLI.

---
## 2026-05-05 — Estrategia de edición desde tabla radicaciones (deuda técnica pendiente)

- **Contexto:** La tabla de radicaciones usa `RadicacionResumen` que no incluye campos de persona (nombres, apellidos, etc.). El botón "Editar" pasa `casoSeleccionado = null` al modal, por lo que el formulario abre vacío incluso con el `effect()` aplicado.
- **Decisión:** No se resuelve en Fase 1. Se deja como tarea futura. La opción preferida es agregar un endpoint `GET /radicaciones/{id}/caso` que retorne el `CasoRadicacionResponse` completo, y llamarlo al abrir el modal en modo editar.
- **Impacto:** back + front
- **Alternativas descartadas:** Enriquecer `RadicacionResumen` — implicaría cambiar el modelo de listado y posiblemente el endpoint de lista, con impacto en performance. Cargar desde la tabla el objeto completo — la tabla no tiene los datos y haría falta un join o segunda consulta de todas formas.

---

---
## 2026-05-05 — Eliminar controller legacy por conflicto de bean

- **Contexto:** Al arrancar el back, Spring lanzaba `ConflictingBeanDefinitionException` porque existían dos clases `RadicacionController` — una en `controller/` (legacy) y otra en `radicacion/adapter/in/web/` (hexagonal). Ambas generaban bean name `radicacionController`.
- **Decisión:** Eliminar el controller legacy (`controller/RadicacionController.java`). El hexagonal ya tiene todos los endpoints del legacy + los nuevos.
- **Impacto:** back
- **Alternativas descartadas:** Renombrar el bean con `@Component("radicacionControllerLegacy")` — agrega complejidad innecesaria y mantiene código muerto.

---
## 2026-05-05 — MongoDB embebido 7.0.4 para Apple Silicon

- **Contexto:** Flapdoodle descargó un binario MongoDB 4.0.2 que es x86-only. En Apple Silicon (ARM64) falla con "Bad CPU type in executable" (error 86).
- **Decisión:** Actualizar `de.flapdoodle.mongodb.embedded.version` de `4.0.2` a `7.0.4` en `application-local.yaml`. MongoDB 7.0+ tiene binarios ARM64 nativos para macOS.
- **Impacto:** back (solo perfil local/test)
- **Alternativas descartadas:** Usar Rosetta 2 para ejecutar el binario x86 — agrega dependencia de emulación y es más lento.

---
## 2026-05-05 — Modal análisis: agregar secciones sin eliminar existentes

- **Contexto:** El agente front reemplazó completamente el modal de análisis (eliminó secciones de visualización de datos del caso). Luis reportó que se "dañó" el modal.
- **Decisión:** Restaurar las secciones de visualización (Asegurado, Radicado, Siniestro, Bancaria, OpenL, IA, Archivos) usando los componentes compartidos existentes, y mantener intactas las secciones de decisión (A-F) que ya estaban. Regla: nunca eliminar funcionalidad existente al agregar nueva.
- **Impacto:** front
- **Alternativas descartadas:** Ninguna — era un bug, no una decisión de diseño.

---
