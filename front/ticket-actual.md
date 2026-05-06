# 🔧 TICKET FRONT — Compactar grillas de modales (radicación + análisis)

**Problema:**
Los modales usan `form-grid` (3 columnas) para todas las secciones. Campos cortos como "CC", "35", "Ahorros" se ven estirados y se pierde mucho espacio visual. Los modales se ven vacíos y poco profesionales.

**Solución:**
Reemplazar `form-grid` por clases `sb-ui-grid-cols-*` + `sb-ui-gap-3` de la librería Seguros Bolívar, eligiendo el número de columnas según el tipo de campos de cada sección.

---

## Regla por tipo de sección

| Tipo de campos | Columnas | Clase |
|---|---|---|
| Campos cortos (tipo doc, edad, código) | 4 | `sb-ui-grid-cols-4 sb-ui-gap-3` |
| Campos medianos (nombre, póliza, entidad) | 3 | `sb-ui-grid-cols-3 sb-ui-gap-3` |
| Campos largos (correo, producto) | 2 | `sb-ui-grid-cols-2 sb-ui-gap-3` |
| Textareas (causa, descripción, observación) | 1 | `sb-ui-grid-cols-1 sb-ui-gap-3` |

---

## Modal Radicación (`modal-crear-caso.component.html`)

### Datos del Asegurado → 3 columnas
```html
<div class="sb-ui-grid-cols-3 sb-ui-gap-3">
  <!-- Tipo Doc, Nro Doc, Nombre, Correo, Teléfono -->
</div>
```

### Información de Póliza → 3 columnas
```html
<div class="sb-ui-grid-cols-3 sb-ui-gap-3">
  <!-- Nro Póliza, Cobertura, Tipo Póliza -->
</div>
```

### Información del Siniestro → 3 columnas para campos cortos + 1 columna para textareas
```html
<div class="sb-ui-grid-cols-3 sb-ui-gap-3">
  <!-- Nro Siniestro, Fecha Siniestro, Ciudad -->
</div>
<div class="sb-ui-grid-cols-1 sb-ui-gap-3 mt-2">
  <!-- Causa (textarea), Descripción (textarea) -->
</div>
```

### Datos Bancarios → 3 columnas
```html
<div class="sb-ui-grid-cols-3 sb-ui-gap-3">
  <!-- Entidad, Nro Cuenta, Tipo Cuenta -->
</div>
```

---

## Modal Análisis (`modal-analisis.component.html`)

### Secciones de visualización (SeccionFormularioDinamicoComponent)
Estas secciones usan `SeccionFormularioDinamicoComponent` que internamente usa `form-grid`. Hay dos opciones:

**Opción 1 (preferida):** Modificar `SeccionFormularioDinamicoComponent` para aceptar un input `columnas` que controle la grilla:
```typescript
// En seccion-formulario-dinamico.component.ts
columnas = input(3); // default 3
```
```html
<!-- En su template, reemplazar form-grid por: -->
<div [class]="'sb-ui-grid-cols-' + columnas() + ' sb-ui-gap-3'">
```

Luego en modal-analisis.html:
```html
<app-seccion-formulario-dinamico [campos]="camposAsegurado" [valores]="datosCaso()" [columnas]="4" />
<app-seccion-formulario-dinamico [campos]="camposRadicado" [valores]="datosCaso()" [columnas]="4" />
<app-seccion-formulario-dinamico [campos]="camposSiniestro" [valores]="datosCaso()" [columnas]="3" />
<app-seccion-formulario-dinamico [campos]="camposBancaria" [valores]="datosCaso()" [columnas]="3" />
<app-seccion-formulario-dinamico [campos]="camposOpenL" [valores]="datosCaso()" [columnas]="3" />
```

**Opción 2 (si no quieres tocar el shared):** Dejar `SeccionFormularioDinamicoComponent` como está (ya usa form-grid de 3 cols) y solo ajustar el modal de radicación.

---

## Archivos a modificar

- `listar-radicaciones/modal-crear-caso/modal-crear-caso.component.html` — reemplazar `form-grid` por `sb-ui-grid-cols-*`
- `shared/components/secciones/seccion-formulario-dinamico/` (.ts y .html) — agregar input `columnas` (Opción 1)
- `listar-analisis/modal-analisis/modal-analisis.component.html` — pasar `[columnas]` a cada sección

## Criterio de aceptación
- [ ] Modal radicación: campos compactos, sin espacio vacío excesivo
- [ ] Modal análisis: secciones con 4 columnas para campos cortos, 3 para medianos
- [ ] Textareas siguen ocupando ancho completo
- [ ] Responsive: en pantallas < 768px colapsa a 1 columna (sb-ui-grid ya lo maneja)
- [ ] Build exitoso
- [ ] No se crea CSS custom — solo clases `sb-ui-grid-cols-*` y `sb-ui-gap-*`

## Depende de
Ninguna
