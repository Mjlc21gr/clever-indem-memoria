# Módulo Análisis — Historial

---
## 2026-05-05
- Endpoints mockeados (cargarAnalisis, guardarDecision, devolver, anular, amparos)
- Modal análisis completo (6 secciones: Decisión, Coberturas, RENTECH, Objeción, Movilizado, Observación)
- 15 causales de objeción implementadas
- Front conectado al backend (listar-analisis, listar-analisis-linea, listar-mis-analisis)

## 2026-05-06
- GET /analisis conectado a MongoDB real (6 colecciones)
- POST /decision persiste en datos_casos (estado + desicionAnalista + coberturas)
- POST /devolver-radicacion persiste observación en datos_casos
- POST /anular persiste en datos_casos
- Schema validation aplicado a datos_casos
---
