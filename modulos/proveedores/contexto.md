# Módulo Proveedores — Contexto

## Flujo
1. Caso llega con estado `P` (movilizado desde análisis)
2. Se asigna a uno o más proveedores: UIFA, Médico, Investigador, Técnico
3. Proveedor gestiona y devuelve resultado
4. Caso vuelve a Análisis o pasa a Autorizado

## Estados en este módulo
- `P` — En gestión de proveedor

## Pendiente de implementación
- Endpoints de asignación y gestión
- Modales de cada proveedor conectados al backend
