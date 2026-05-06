# Módulo Radicación — Endpoints

## GET /siniestros/api/v1/radicaciones?estado=R&page=0&size=10
Lista radicaciones en estado R (radicado). Paginado.

## GET /siniestros/api/v1/casos/{idRadicado}/analisis
Carga datos completos del caso (6 colecciones). Devuelve Map<String, String>.

## PATCH /siniestros/api/v1/radicaciones/{idRadicado}/siniestro
Asigna número de siniestro. Cambia estado R → A.
Body: `{ "numero_siniestro": "12345" }`
Validación: @NotBlank, @Pattern("\\d+")

## POST /siniestros/api/v1/radicaciones/{idRadicado}/crear-caso
Crea caso asociado a radicación existente. Valida que exista (404 si no).
