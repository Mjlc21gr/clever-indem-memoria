# Módulo Radicación — Historial

---
## 2026-05-05
- Endpoints mockeados creados (crear, guardar siniestro, editar)
- Front conectado al backend real (listar radicaciones estado=R)
- Modal crear/editar caso implementado

## 2026-05-06
- Modal reestructurado: carga datos reales via GET /casos/{id}/analisis
- Modo ver (readonly) y editar (solo siniestro editable)
- Botón "Guardar Siniestro" eliminado → un solo botón "Guardar"
- Alertas sb-ui-alert (warning si falta siniestro, success al guardar)
- Validación: solo dígitos en número de siniestro
- Schema validation MongoDB aplicado a colección radicaciones
- DataInitializer: casos RR/RS sin número de siniestro (para probar flujo)
---
