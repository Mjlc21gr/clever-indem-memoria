---
inclusion: always
---

# Estructura de Archivos — Clever Indem Frontend

## Raíz del Proyecto

```
clever-frontend/
├── angular.json                       # Configuración Angular CLI (proyecto: indem-clever-mfe)
├── package.json                       # Dependencias
├── tsconfig.json                      # TypeScript base (paths: @shared)
├── tailwind.config.js                 # Tailwind (content: projects/indem-clever/src/**/*.{html,ts})
├── jest.config.js                     # Jest config
├── setup-jest.ts                      # Setup de Jest
├── .kiro/steering/                    # Reglas y contexto para Kiro CLI
│   ├── rule-project-structure.md      # Regla de organización de componentes
│   └── clever-indem/front/            # ← Este directorio de documentación
└── projects/
    └── indem-clever/                  # Código fuente del MFE
        ├── webpack.config.js          # Webpack Module Federation (dev)
        ├── webpack.prod.config.js     # Webpack Module Federation (prod)
        ├── proxy.conf.json            # Proxy dev → backend
        ├── tsconfig.app.json          # TS config app
        └── src/
            ├── main.ts                # Bootstrapping
            ├── bootstrap.ts           # Bootstrap real (para MFE lazy)
            ├── index.html             # HTML shell
            ├── styles.scss            # Estilos globales
            ├── tailwind.css           # Tailwind entrada
            ├── assets/
            │   └── setenv.ts          # Script de env variables (ts-node)
            └── environments/
                ├── environment.ts     # Dev
                └── environment.prod.ts # Prod
```

## Código fuente (`src/app/`)

```
src/app/
├── app.ts                             # Componente raíz (layout: topbar + sidebar + router-outlet)
├── app.html                           # Template del layout principal
├── app.scss                           # Estilos del layout
├── app.config.ts                      # ApplicationConfig (providers globales)
├── app.routes.ts                      # Rutas raíz (incluye REMOTE_ROUTES + error pages)
├── exposed.routes.ts                  # Rutas expuestas vía Module Federation
│
├── config/
│   └── environment.ts                 # AppEnvironment interface + instancia dev
│
├── core/
│   ├── guards/
│   │   ├── auth.guard.ts              # authGuard — verifica isAuthenticated
│   │   └── role.guard.ts              # roleGuard — verifica rol desde route.data.roles
│   ├── interceptors/
│   │   └── auth.interceptor.ts        # HTTP interceptor: 401→inicio, 403→acceso-denegado, 500→toast
│   ├── models/
│   │   ├── index.ts                   # Barrel export de todos los modelos
│   │   ├── usuario.model.ts           # Usuario, RolUsuario
│   │   ├── radicacion.model.ts        # Radicacion, RadicacionResumen
│   │   ├── analisis.model.ts          # Analisis
│   │   ├── proveedor.model.ts         # CasoProveedor, TipoProveedor
│   │   ├── orden-pago.model.ts        # OrdenPago, Objecion
│   │   ├── caso-consulta.model.ts     # CasoConsulta, GestionHistorial, EtapaHistorial, TiempoResumen
│   │   └── api-response.model.ts      # ApiResponse<T>, PaginatedData<T>
│   ├── pages/
│   │   ├── acceso-denegado/
│   │   │   └── acceso-denegado.component.ts   # Página 403
│   │   └── no-encontrado/
│   │       └── no-encontrado.component.ts     # Página 404
│   └── services/
│       ├── auth.service.ts            # AuthService (stub JWT) — signal-based
│       ├── notificacion.service.ts    # NotificacionService (toasts) — signal-based
│       └── radicaciones.service.ts    # RadicacionesService — HTTP real al backend
│
├── features/
│   └── indem-clever/
│       └── components/
│           ├── inicio/                        # /inicio — Dashboard gerencial
│           ├── listar-radicaciones/           # /listar-radicaciones
│           │   ├── modal-agregar-caso/        # Modal: agregar/ver radicado (3 tabs)
│           │   │   ├── ventana-datos-generales/
│           │   │   └── ventana-data-operativa/
│           │   ├── modal-mesa-perfeccionamiento/
│           │   └── modal-radicacion/
│           ├── listar-mesa-perfect/           # /listar-mesa-perfect
│           ├── listar-analisis/               # /listar-analisis
│           │   └── modal-analisis/            # Modal compartido por analisis/linea/mis
│           ├── listar-analisis-linea/         # /listar-analisis-linea
│           ├── listar-mis-analisis/           # /listar-mis-analisis (reutiliza modal-analisis)
│           ├── listar-uifa/                   # /listar-uifa
│           │   └── modal-uifa/
│           ├── listar-medico/                 # /listar-medico
│           │   └── modal-medico/
│           ├── listar-investigador/           # /listar-investigador
│           │   └── modal-investigador/
│           ├── listar-tecnico/                # /listar-tecnico
│           │   └── modal-tecnico/
│           ├── listar-pagos/                  # /listar-pagos (solo admin)
│           │   └── modal-pago/
│           ├── listar-casos/                  # /listar-casos
│           │   └── modal-caso/
│           └── listar-usuarios/               # /listar-usuarios (solo admin)
│
└── shared/
    ├── public-api.ts                   # Barrel export de todo shared (accedido por @shared)
    ├── models/
    │   └── sidebar-menu.model.ts       # SidebarMenuGroup, SidebarMenuItem
    ├── data/
    │   └── sidebar-menu.data.ts        # SIDEBAR_MENU — configuración del menú lateral
    ├── mocks/
    │   ├── index.ts
    │   ├── mock-radicaciones.data.ts   # MOCK_RADICACIONES
    │   ├── mock-analisis.data.ts       # MOCK_ANALISIS
    │   ├── mock-proveedores.data.ts    # MOCK_UIFA, MOCK_MEDICO, MOCK_INVESTIGADOR, MOCK_TECNICO
    │   └── mock-pagos.data.ts          # MOCK_DECISIONES (+ MOCK_ORDENES_PAGO, MOCK_OBJECIONES deprecados)
    └── components/
        ├── topbar/                     # TopbarComponent — barra superior
        ├── sidebar/                    # SidebarComponent — menú lateral
        ├── tabla-dinamica/             # TablaDinamicaComponent — tabla reutilizable
        ├── boton-accion/               # BotonAccionComponent — botón sb-ui
        ├── dialogo-confirmacion/       # DialogoConfirmacionComponent — modal confirm
        └── secciones/                  # Secciones reutilizables para modales
            ├── seccion-formulario-dinamico/   # Formulario con campos configurables
            ├── seccion-observaciones-generica/ # Textarea con section-card
            ├── seccion-panel-generico/        # Panel colapsable genérico
            ├── seccion-archivos/              # File upload
            ├── seccion-consulta/              # Consultar data operativa
            ├── seccion-decision/              # Decisión del analista (select estado/causal)
            ├── seccion-derivacion/            # Derivación a proveedores (checkboxes)
            ├── seccion-analisis-ia/           # Análisis IA (hospitalización, cirugía, incapacidad)
            ├── seccion-checklist/             # Checklist de documentos
            └── seccion-linea-tiempo/          # Historial stepper + acordeón
```

## Regla de estructura de componentes

Ver `.kiro/steering/rule-project-structure.md` para la regla completa.

**Resumen**: Modal → dentro del componente que lo invoca. Tab del modal → `ventana-*`. Fieldset de ventana → `secciones/seccion-*`. Componente reutilizable → `shared/components/`.
