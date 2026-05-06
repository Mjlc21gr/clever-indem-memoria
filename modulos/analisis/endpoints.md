# Módulo Análisis — Endpoints

## GET /siniestros/api/v1/radicaciones?estado=A&page=0&size=50
Lista casos en estado A (análisis). Paginado.

## GET /siniestros/api/v1/casos/{idRadicado}/analisis
Carga datos completos del caso (6 colecciones). Devuelve Map<String, String>.

## POST /siniestros/api/v1/casos/{idRadicado}/decision
Registra decisión del analista. Cambia estado según tipo_decision.
Body: `DecisionAnalisisRequest` (tipo_decision, coberturas, causal, observación)

## POST /siniestros/api/v1/casos/{idRadicado}/devolver-radicacion
Devuelve caso a radicación. Estado → R.
Body: `{ "observacion": "..." }`

## POST /siniestros/api/v1/casos/{idRadicado}/anular
Anula caso. Estado → AN.
Body: `{ "observacion": "..." }`

## GET /siniestros/api/v1/amparos
Catálogo de 11 amparos disponibles.
