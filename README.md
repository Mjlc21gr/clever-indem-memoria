# Clever Indem — Memoria Compartida

Repositorio de contexto y memoria del proyecto **Clever Indem** (Seguros Bolívar).  
Kiro CLI lo lee automáticamente desde `~/.kiro/steering/clever-indem/`.

## Cómo usar

### Setup inicial (cada dev)
```bash
cd ~/.kiro/steering/
git clone git@github.com:segurosbolivar/clever-indem-memoria.git clever-indem
```

### Al iniciar sesión de trabajo
```bash
cd ~/.kiro/steering/clever-indem && git pull
```

### Al terminar sesión de trabajo
```bash
cd ~/.kiro/steering/clever-indem
git add .
git commit -m "sesión YYYY-MM-DD — [back|front|auditor] — resumen breve"
git push
```

## Estructura

```
clever-indem/
├── README.md                    # Este archivo
│
├── estado-actual.md             # Foto del proyecto HOY (solo auditor edita)
├── tareas.md                    # Log de todas las tareas (append-only)
├── decisiones.md                # Decisiones técnicas (append-only)
│
├── back/                        # Contexto técnico del backend
│   ├── architecture.md
│   ├── api-contracts.md
│   ├── domain-model.md
│   ├── persistence.md
│   ├── config-beans.md
│   ├── mappers.md
│   ├── ia-integration.md
│   ├── testing-strategy.md
│   └── ticket-actual.md        # Ticket en curso (se sobreescribe)
│
├── front/                       # Contexto técnico del frontend
│   ├── 01-overview.md
│   ├── 02-structure.md
│   ├── 03-routing.md
│   ├── 04-models.md
│   ├── 05-components-shared.md
│   ├── 06-components-features.md
│   ├── 07-services.md
│   ├── 08-styles.md
│   ├── 09-patterns.md
│   ├── 10-mocks-backend.md
│   ├── 11-layout.md
│   └── ticket-actual.md        # Ticket en curso (se sobreescribe)
│
├── modulos/                     # Memoria por módulo de negocio
│   ├── radicacion/
│   │   ├── contexto.md          # Flujo, reglas, estados
│   │   ├── endpoints.md         # Contratos API
│   │   └── historial.md         # Qué se hizo (append-only)
│   ├── analisis/
│   │   ├── contexto.md
│   │   ├── endpoints.md
│   │   └── historial.md
│   ├── proveedores/
│   │   ├── contexto.md
│   │   ├── endpoints.md
│   │   └── historial.md
│   ├── autorizante/
│   │   ├── contexto.md
│   │   ├── endpoints.md
│   │   └── historial.md
│   └── casos/
│       ├── contexto.md
│       ├── endpoints.md
│       └── historial.md
│
├── auditor/                     # Instrucciones del auditor
│   └── CLAUDE.md
│
└── resultados/                  # Output de cada sesión (append-only)
    └── YYYY-MM-DD-[back|front]-descripcion.md
```

## Reglas

1. **Archivos append-only** (`tareas.md`, `decisiones.md`, `historial.md`, `resultados/`): nunca editar entradas anteriores, solo agregar al final.
2. **Archivos overwrite** (`estado-actual.md`, `ticket-actual.md`): se sobreescriben cada sesión.
3. **Contexto técnico** (`back/`, `front/`): se actualiza cuando cambia la arquitectura.
4. **Módulos** (`modulos/`): cada dev actualiza el historial del módulo en el que trabajó.
5. **Commits**: un commit por sesión con formato `"sesión YYYY-MM-DD — [rol] — resumen"`.

## Cómo retomar contexto — Prompts por rol

### Auditor (coordinador)
```
Lee estos archivos para retomar el contexto completo:
- ~/.kiro/steering/clever-indem/estado-actual.md
- ~/.kiro/steering/clever-indem/tareas.md
- ~/.kiro/steering/clever-indem/decisiones.md
- ~/.kiro/steering/clever-indem/back/ticket-actual.md
- ~/.kiro/steering/clever-indem/front/ticket-actual.md

Cuando tengas todo el contexto dime dónde quedamos y qué sigue.
```

### Back (desarrollador backend)
```
Eres el desarrollador back del proyecto clever-indem.
Lee ~/.kiro/steering/clever-indem/back/ticket-actual.md y ejecuta todo lo que dice.
Al terminar, actualiza ~/.kiro/steering/clever-indem/resultados/ con un archivo YYYY-MM-DD-back-descripcion.md describiendo todo lo que hiciste.
Luego responde: "Listo. Puedes pedirle al auditor que verifique."
```

**Inicio de sesión back (primera vez o retomar):**
```
Eres el desarrollador back del proyecto clever-indem.
Lee ~/.kiro/steering/clever-indem/back/ para entender la arquitectura.
Luego lee la estructura del repo en /Users/luiscarlosgomez/GitHub/Clever Indem/clever-siniestros-ms
Rama de trabajo: feature/autorizante
Cuando tengas el contexto completo confirma que estás listo.
```

**Comandos del back:**
```bash
cd "/Users/luiscarlosgomez/GitHub/Clever Indem/clever-siniestros-ms"
git checkout feature/autorizante
./gradlew bootRun
```

### Front (desarrollador frontend)
```
Eres el desarrollador front del proyecto clever-indem.
Lee ~/.kiro/steering/clever-indem/front/ticket-actual.md y ejecuta todo lo que dice.
Al terminar, actualiza ~/.kiro/steering/clever-indem/resultados/ con un archivo YYYY-MM-DD-front-descripcion.md describiendo todo lo que hiciste.
Luego responde: "Listo. Puedes pedirle al auditor que verifique."
```

**Inicio de sesión front (primera vez o retomar):**
```
Eres el desarrollador front del proyecto clever-indem.
Lee ~/.kiro/steering/clever-indem/front/ para entender la arquitectura.
Luego lee la estructura del repo en /Users/luiscarlosgomez/GitHub/Clever Indem/clever-frontend
Rama de trabajo: feature/front-servicios
Cuando tengas el contexto completo confirma que estás listo.
```

**Comandos del front:**
```bash
cd "/Users/luiscarlosgomez/GitHub/Clever Indem/clever-frontend"
git checkout feature/front-servicios
npm start
```
