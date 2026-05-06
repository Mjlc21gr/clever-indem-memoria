# Módulo Análisis — Contexto

## Flujo
1. Caso llega con estado `A` (desde radicación)
2. Analista revisa datos del caso (asegurado, póliza, siniestro, bancaria, IA)
3. Toma decisión: PAGAR, OBJETAR, MOVILIZAR o ANULAR
4. Según decisión → cambia estado: AU (autorizado), P (proveedor), AN (anulado), R (devuelto)

## Estados en este módulo
- `A` — En Análisis (visible en tablas de análisis, línea de negocio, mis casos)

## Decisiones posibles
- **PAGAR/OBJETAR** → estado AU (Autorizado)
- **MOVILIZAR** → estado P (Proveedor)
- **ANULAR** → estado AN (Anulado)
- **DEVOLVER** → estado R (vuelve a Radicación)

## Secciones del modal de análisis
- A: Tomar Decisión (tipo + amparo)
- B: Coberturas a Afectar (hospitalización, incapacidad, cirugía)
- C: RENTECH (concepto, diagnóstico, condiciones)
- D: Objeción (15 causales con campos dinámicos)
- E: Movilizado (UIFA, Investigador, Médico, Técnico)
- F: Observación del Analista

## Colecciones MongoDB involucradas
- `radicaciones` — estado principal
- `datos_casos` — estado interno + decisión + coberturas
- `datos_cliente` — datos personales (readonly)
- `datos_ia` — resultados del análisis IA (readonly)
- `datos_bancarios` — info bancaria (readonly)
- `datos_docs` — documentos adjuntos (readonly)
