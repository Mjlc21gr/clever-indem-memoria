# Módulo Autorizante — Contexto

## Flujo
1. Caso llega con estado `AU` (autorizado desde análisis)
2. Autorizante revisa y aprueba/devuelve la orden de pago u objeción
3. Si aprueba → estado C (cerrado)
4. Si devuelve → vuelve a Análisis

## Estados en este módulo
- `AU` — Autorizado (pendiente de aprobación)

## Pendiente de implementación
- Endpoints de aprobación/devolución
- Conexión de listar-pagos al backend
