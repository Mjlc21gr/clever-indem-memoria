# Estado actual — clever-indem
_Última actualización: 2026-05-05 23:18_

## Resumen ejecutivo
Flujo funcional end-to-end implementado. Back persiste cambios de estado en MongoDB. Front consume backend real en radicación (estado=R) y análisis (estado=A). Modal de análisis completo con RENTECH ampliado. Los casos se mueven entre módulos al cambiar estado.

## Back (`feature/autorizante`) — CORRIENDO, flujo funcional
- ✅ Cambios de estado persisten en MongoDB (R→A→AU/P/AN→R)
- ✅ Enum EstadoRadicacion ampliado: R, A, AU, AN, I, P, C
- ✅ RadicacionService.actualizarEstado() — busca, valida, actualiza, guarda
- ✅ PATCH /radicaciones/{id}/siniestro → estado R→A (persiste)
- ✅ POST /casos/{id}/decision → PAGAR/OBJETAR→AU, MOVILIZAR→P, ANULAR→AN (persiste)
- ✅ POST /casos/{id}/devolver-radicacion → estado→R (persiste)
- ✅ POST /casos/{id}/anular → estado→AN (persiste)
- ✅ POST /radicaciones/{id}/crear-caso → valida existencia (404 si no existe)
- ✅ GET /amparos → catálogo 11 items
- ✅ GET /casos/{id}/analisis → mock
- 🟡 RadicacionControllerTest falla por ${api.paths.prefix} no resuelto en @WebMvcTest
- 🟡 Service legacy huérfano (service/RadicacionServiceImpl)

## Front (`feature/front-servicios`) — BUILD SUCCESSFUL
- ✅ listar-radicaciones: consume backend (estado=R), refresh tras guardar siniestro
- ✅ listar-analisis: consume backend (estado=A), refresh tras decisión
- ✅ modal-analisis completo:
  - Visualización: Asegurado, Radicado, Siniestro, Bancaria, OpenL, IA, Archivos
  - Sección A: Tomar Decisión (PAGAR/OBJETAR/MOVILIZAR/ANULAR + amparo)
  - Sección B: Coberturas a Afectar (3 tipos con cálculos automáticos)
  - Sección C: RENTECH ampliado (13 campos: diagnóstico, pernocto, resúmenes auto, condiciones)
  - Sección D: Objeción (15 causales con campos dinámicos)
  - Sección E: Movilizado (4 opciones con observaciones)
  - Sección F: Observación del Analista (auto-generada + manual)
- ✅ modal-crear-caso: crear/editar con sección siniestro
- ✅ Proxy corregido: apunta a localhost:8081

## Bloqueantes activos
- Ninguno

## Próximos pasos
1. **[BACK] Conectar GET /analisis a MongoDB real + persistir decisiones** ← EN CURSO
2. Prueba manual completa end-to-end en navegador
3. Conectar módulos de proveedores (UIFA, Médico, Investigador, Técnico)
4. Conectar listar-pagos al backend
