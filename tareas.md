# Log de tareas — clever-indem

---
## 2026-05-05 — Sesión principal

### [BACK] Servicios mockeados Radicación + Análisis
- **Estado:** completado
- **Ticket:** Renombrar consultarPorId → consultarRadicacionPorId. Crear endpoints mockeados: crearCasoRadicacion (POST), guardarSiniestro (PATCH), editarCasoRadicacion (PUT), cargarAnalisis (GET), listarAmparos (GET), guardarDecision (POST), devolverRadicacion (POST), anularCaso (POST)
- **Resultado:** BUILD SUCCESSFUL. 9 endpoints mockeados funcionando. RadicacionController actualizado. AnalisisController + AmparoController creados en analisis/adapter/in/web/.
- **Auditoría:** 🟡 DecisionAnalisisRequest usa camelCase+@JsonProperty en vez de snake_case nativo (deuda técnica, no bloqueante). 🟡 Controllers analisis/ sin capa application/domain/ (deuda técnica aceptada hasta MongoDB).

### [FRONT] Módulo Radicación + Modal Análisis
- **Estado:** completado con bugs
- **Ticket:** Alinear tabla radicaciones con tabla análisis, botón Crear Caso, modal creación/edición con sección siniestro, servicio CasosRadicacionService, servicio AnalisisService, modal análisis completo (secciones: Tomar Decisión, Coberturas, RENTECH, Objeción, Movilizado, Observación)
- **Resultado:** Servicios y modelos correctos. 2 bugs bloqueantes + 2 importantes.
- **Auditoría:** 🔴 ngOnChanges incompatible con signal inputs (ModalAnalisis no carga datos). 🔴 interface Radicacion duplicada (no compila). 🟡 ModalCrearCaso no pre-llena campos en editar. 🟡 ModalAnalisis no resetea estado entre casos.

### [FRONT] Fix bugs bloqueantes post-auditoría
- **Estado:** completado
- **Ticket:** 1) Reemplazar ngOnChanges por effect() en ModalAnalisisComponent. 2) Eliminar interface Radicacion duplicada en radicacion.model.ts. 3) Agregar effect() para pre-llenar campos en ModalCrearCasoComponent. 4) Agregar resetForm() en ModalAnalisisComponent.
- **Resultado:** BUILD SUCCESSFUL. 0 errores. Los 4 bugs corregidos. Solo warnings NG8102 no bloqueantes.
- **Auditoría:** ✅ Todos los fixes aplicados correctamente. 🔵 Modo editar sigue abriendo vacío porque RadicacionResumen no tiene campos de persona — requiere estrategia separada (Fase 2).

---
## 2026-05-05 00:00

### [AUDITOR] Inicialización del scope
- **Estado:** completado
- **Ticket:** Sesión inicial — se registraron repos, ramas y stack en config.md
- **Resultado:** scope/ creado con config.md, estado-actual.md, tareas.md, decisiones.md, comunicacion-agentes.md
- **Auditoría:** ninguna
---

---
## 2026-05-05 22:40

### [BACK] Fix conflicto de bean + MongoDB ARM64
- **Estado:** completado
- **Ticket:** Eliminar controller legacy que causaba ConflictingBeanDefinitionException. Actualizar MongoDB embebido a 7.0.4 para Apple Silicon.
- **Resultado:** Back arranca correctamente en localhost:8081. Endpoints responden.
- **Auditoría:** ✅ Sin problemas. Service legacy (`service/RadicacionServiceImpl`) queda huérfano pero no causa conflicto — limpiar en fase de eliminación de paquete legacy.

### [FRONT] Conectar tabla análisis al backend + restaurar modal
- **Estado:** completado
- **Ticket:** 1) listar-analisis: reemplazar MOCK_ANALISIS por llamada a RadicacionesService.listar({estado:'A'}). 2) modal-analisis: restaurar secciones de visualización (SeccionFormularioDinamicoComponent x5, SeccionAnalisisIaComponent, SeccionArchivosComponent) antes de las secciones de decisión existentes.
- **Resultado:** Build exitoso. Tabla consume backend real (2 registros en estado A). Modal tiene secciones de visualización + decisión completas.
- **Auditoría:** ✅ Código verificado. Imports correctos, campos readonly definidos, HTML insertado en posición correcta.

---

---
## 2026-05-06 08:44

### [FRONT] Corregir causales de objeción en modal-analisis
- **Estado:** completado
- **Ticket:** Actualizar causalesObjecionOpciones (15 opciones exactas), corregir @case del @switch en HTML, agregar limpieza de campos al cambiar causal.
- **Resultado:** Build exitoso. 15 causales con campos correctos. Limpieza al cambiar causal implementada.
- **Auditoría:** ✅ Aprobado. Todos los @case coinciden 1:1 con la especificación de negocio.
---

---
## 2026-05-06 09:03

### [BACK] Fix RadicacionControllerTest + Eliminar paquete legacy
- **Estado:** completado
- **Ticket:** 1) Agregar @TestPropertySource al RadicacionControllerTest. 2) Eliminar directorios legacy: controller/, service/, repository/, domain/.
- **Resultado:** Build exitoso. 3 tests pasan. 24 archivos legacy + 1 test legacy eliminados. Paquetes hexagonales intactos.
- **Auditoría:** ✅ Aprobado. Código limpio, sin conflictos de bean.
---

---
## 2026-05-06 09:09

### [FRONT] Conectar listar-analisis-linea y listar-mis-analisis al backend
- **Estado:** completado
- **Ticket:** Reemplazar MOCK_ANALISIS por RadicacionesService.listar({estado:'A'}) en ambos componentes.
- **Resultado:** Build exitoso. Ambos consumen backend real. Modal conectado con casoId y refresh.
- **Auditoría:** ✅ Aprobado. MOCK_ANALISIS eliminado completamente del proyecto.
---

---
## 2026-05-06 09:48

### [BACK] Implementar modelo de datos real MongoDB (6 colecciones)
- **Estado:** completado
- **Ticket:** Crear bounded context siniestro/ con 6 colecciones reales.
- **Resultado:** 38 archivos creados. Build exitoso. SiniestroService con listarCasos() y obtenerDetalle(). DataInitializer con 5 casos. BeanConfiguration actualizado.
- **Auditoría:** ✅ Aprobado. Arquitectura hexagonal correcta, BigDecimal para monetarios, typos de negocio respetados.
---

---
## 2026-05-06 10:03

### [BACK] Conectar a MongoDB local (siniestro_db) en vez de Flapdoodle
- **Estado:** completado
- **Ticket:** Cambiar perfil local para usar mongodb://localhost:27017/siniestro_db.
- **Resultado:** URI cambiada, Flapdoodle movido a testImplementation, tests siguen con embebido 7.0.4.
- **Auditoría:** ✅ Aprobado.
---

---
## 2026-05-06 10:27

### [BACK] Poblar MongoDB con 120 casos realistas
- **Estado:** pendiente
- **Ticket:** Reescribir DataInitializer para insertar 120 casos con datos realistas en las 6 colecciones. Distribución: RR(30), RS(15), A(25), AU(15), AN(15), P(10), C(10). Datos basados en ejemplo real de producción. deleteAll + saveAll en cada arranque.
- **Resultado:** pendiente
- **Auditoría:** pendiente
---

---
## 2026-05-06 10:28

### [BACK] Poblar MongoDB con 120 casos realistas
- **Estado:** completado
- **Ticket:** Reescribir DataInitializer con 120 casos en 6 colecciones
- **Resultado:** 120 casos insertados. Verificado con mongosh: radicaciones(120), datos_casos(120), datos_cliente(120), datos_ia(50), datos_docs(240), datos_bancarios(120)
- **Auditoría:** ✅ Aprobado
---

---
## 2026-05-06 10:47

### [BACK] Conectar GET /analisis a MongoDB real + persistir decisiones
- **Estado:** pendiente
- **Ticket:** Ver detalle en ~/.kiro/scope/ticket-back-analisis-real.md
- **Resultado:** pendiente
- **Auditoría:** pendiente
---

---
## 2026-05-06 11:10

### [FRONT] Reestructurar modal ver/editar en radicaciones
- **Estado:** completado
- **Ticket:** Modal carga datos reales del backend via GET /casos/{id}/analisis. Modo ver (readonly) y editar (solo nro siniestro editable). Un solo botón Guardar: valida siniestro → alerta si vacío, guarda y cambia estado a A si tiene valor. UI con 4 secciones (Asegurado, Póliza, Siniestro, Bancaria). Eliminado botón "Guardar Siniestro" separado.
- **Resultado:** Build exitoso. Modal reestructurado con carga desde backend.
- **Auditoría:** ✅ Aprobado.
---
