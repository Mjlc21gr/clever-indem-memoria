# Módulo Radicación — Contexto

## Flujo
1. Caso llega al sistema → estado `R` (Radicado)
2. Analista abre el caso, revisa datos del asegurado/póliza/siniestro
3. Si tiene número de siniestro → Guardar → cambia a estado `A` (Análisis)
4. Si no tiene siniestro → alerta warning, no avanza

## Estados en este módulo
- `R` — Radicado (visible en tabla de radicaciones)

## Colecciones MongoDB involucradas
- `radicaciones` — registro principal (id_radicado, poliza, estado)
- `datos_casos` — estado interno del caso (RR/RS)
- `datos_cliente` — datos personales + siniestro + bancarios embebidos

## Reglas de negocio
- No se puede pasar a Análisis sin número de siniestro
- El número de siniestro solo acepta dígitos
- Al guardar siniestro, el estado cambia de R → A en `radicaciones` y de RR → A en `datos_casos`
