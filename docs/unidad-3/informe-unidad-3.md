# 1. Modelado de Procesos
El Modelado de Procesos representa la evolución lógica de los casos de uso y macrofunciones definidas hacia Diagramas de Flujo de Datos (DFD) acoplados directamente con la base de datos relacional y con los servicios de integración confirmados por ContaQuick PRO. A diferencia de los diagramas de flujo operativos previos, los DFDs demuestran con precisión de dónde lee la información cada proceso, qué reglas o cálculos aplica, en qué tablas persiste los cambios y qué salidas produce para cada actor.

## 1.1. Catálogo de Almacenes de Datos (D1 a D9)

Para garantizar trazabilidad estricta entre los diagramas de procesos de esta unidad y el Modelo Entidad-Relación (DER) normalizado en 4FN de la Unidad II, todos los almacenes de datos referenciados en los DFDs se estandarizan de la siguiente manera:

| Código DFD | Nombre del Almacén de Datos | Tablas Físicas / Servicio de Almacenamiento Asociado | Esquema / Ubicación |
|---|---|---|---|
| **D1** | Maestro de Empresas y Cuentas | `global_empresas` (horas_mensuales_plan), `global_cuentas_maestras` | `ContaQuickDBMaster` |
| **D2** | Seguridad y Permisos RBAC | `tbl_usuarios`, `tbl_roles`, `tbl_usuarios_roles`, `tbl_objetos`, `tbl_permisos` | `soporte` (PostgreSQL) |
| **D3** | Catálogos de Soporte | `tbl_categorias_ticket` (24 módulos ERP), `tbl_tipos_ticket`, `tbl_estados_ticket` | `soporte` (PostgreSQL) |
| **D4** | Repositorio de Tickets | `tbl_tickets` (folio_ticket, ticket_padre_id, tarea_id, horas_invertidas, fecha_creacion, fecha_cierre) | `soporte` (PostgreSQL) |
| **D5** | Adjuntos y Almacenamiento de Objetos | `tbl_ticket_adjuntos` (metadatos) + Cloudflare R2 (archivos S3 con retención 90–180 días) | `soporte` + Cloudflare R2 |
| **D6** | Historial de Tickets | `tbl_ticket_historial` (transiciones de estado y comentarios) | `soporte` (PostgreSQL) |
| **D7** | Backlog y Tablero de Tareas | `tbl_tareas_desarrollo` (codigo_tarea, estado_kanban, usuario_responsable_id) | `soporte` (PostgreSQL) |
| **D8** | Centro de Notificaciones (Campanita) | `control.notificaciones` (tabla existente consumida vía `POST /api/v1/notificaciones`) | `ContaQuickDBMaster` (control) |
| **D9** | Bitácoras de Auditoría Forense | `tbl_bitacora_auditoria` (JSONB), `control.auditoria_global`, `auditoria.log_auditoria` | `soporte` / `ContaQuickDBMaster` |

## 1.2. Diagrama de Flujo de Datos de Contexto (DFD Nivel 0)
El DFD de Contexto muestra el sistema como un único proceso central (Proceso 0) rodeado de los cuatro agentes externos oficiales (AC-01 a AC-04), evidenciando el intercambio de flujos mediante API REST con tokens JWT y el consumo del endpoint interno de notificaciones.
```mermaid
flowchart LR
    %% Agentes Externos (Notación DFD)
    E1["AC-01: Cliente<br/>(Empresa Suscrita)"]
    E2["AC-02: Técnico / Desarrollador<br/>(Equipo de 3 Integrantes)"]
    E3["AC-03: Administrador / Gerencia"]
    E4["AC-04: API Core ContaQuick PRO<br/>(Auth JWT y Campanita)"]

    %% Proceso 0 (Sistema Completo)
    P0(("0<br/>Sistema de Gestión de<br/>Tickets y Tareas<br/>(ContaQuick PRO)"))

    %% Flujos de Datos AC-01
    E1 -->|"Solicitud de ticket (módulo ERP,<br/>asunto, descripción, capturas) y consulta"| P0
    P0 -->|"Folio de ticket, estado actual,<br/>historial de cambios y avisos"| E1

    %% Flujos de Datos AC-02
    E2 -->|"Triaje (tipo), enlace padre-hijo,<br/>tareas Kanban y horas_invertidas"| P0
    P0 -->|"Cola balanceada de tickets,<br/>tablero Kanban/Lista y métricas"| E2

    %% Flujos de Datos AC-03
    E3 -->|"Parámetros RBAC, reasignación<br/>manual y filtros de reportes"| P0
    P0 -->|"Reportes R-01 a R-04, KPIs y<br/>alertas de exceso de plan"| E3

    %% Flujos de Datos AC-04
    P0 -->|"Petición de sesión y POST<br/>/api/v1/notificaciones (Secret Key)"| E4
    E4 -->|"Token JWT (usuarioId, rol, empresaId)<br/>y confirmación de aviso en campanita"| P0

    classDef entity fill:#f8fafc,stroke:#0f172a,stroke-width:2px,color:#0f172a,font-weight:bold;
    classDef extSys fill:#eff6ff,stroke:#1d4ed8,stroke-width:2px,color:#1e3a8a,font-weight:bold;
    classDef process fill:#e0f2fe,stroke:#0369a1,stroke-width:3px,color:#0c4a6e,font-weight:bold;
    class E1,E2,E3 entity;
    class E4 extSys;
    class P0 process;
```

## 1.3. Diagrama de Descomposición Funcional
El Diagrama de Descomposición Funcional desglosa jerárquicamente el Proceso 0 en los 7 procesos principales derivados de las macrofunciones (MF-01 a MF-07) y en sus 19 procesos elementales ejecutables.
```mermaid
flowchart TB
    ROOT["<b>0. Sistema de Gestión de Tickets y Tareas — ContaQuick PRO</b>"]

    P1["<b>1.0 Seguridad, Identidad<br/>y Acceso RBAC</b><br/><i>(MF-01 | CU-01)</i>"]
    P2["<b>2.0 Recepción y Gestión<br/>de Tickets</b><br/><i>(MF-02 | CU-02, CU-03, CU-05)</i>"]
    P3["<b>3.0 Balanceo de Carga<br/>y Enrutamiento</b><br/><i>(MF-03 | CU-04)</i>"]
    P4["<b>4.0 Gestión de Tareas<br/>de Desarrollo</b><br/><i>(MF-04 | CU-08, CU-09)</i>"]
    P5["<b>5.0 Cierre, Horas y<br/>Control de Planes</b><br/><i>(MF-05 | CU-06, CU-07)</i>"]
    P6["<b>6.0 Notificaciones y<br/>Auditoría Forense</b><br/><i>(MF-06 | Transversal)</i>"]
    P7["<b>7.0 Analítica, Reportes<br/>y Retención</b><br/><i>(MF-07 | CU-10)</i>"]

    ROOT --> P1
    ROOT --> P2
    ROOT --> P3
    ROOT --> P4
    ROOT --> P5
    ROOT --> P6
    ROOT --> P7

    P1 --> P11["1.1 Validar JWT vía API Core"]
    P1 --> P12["1.2 Evaluar Permisos RBAC"]
    P1 --> P13["1.3 Administrar Roles y Objetos"]

    P2 --> P21["2.1 Registrar Ticket y Validar Módulos"]
    P2 --> P22["2.2 Almacenar Adjuntos en R2"]
    P2 --> P23["2.3 Consultar Estado e Historial"]
    P2 --> P24["2.4 Clasificar Tipo y Agrupar Duplicados"]

    P3 --> P31["3.1 Calcular Carga Activa de Técnicos"]
    P3 --> P32["3.2 Asignar Automáticamente / Reasignar"]

    P4 --> P41["4.1 Crear y Vincular Tarea 1:N"]
    P4 --> P42["4.2 Actualizar Avance Kanban / Lista"]

    P5 --> P51["5.1 Registrar Horas Invertidas Manuales"]
    P5 --> P52["5.2 Sellar Cierre y Tiempo Resolución"]
    P5 --> P53["5.3 Auditar Saldo vs. Plan Mensual"]

    P6 --> P61["6.1 Despachar Aviso a Campanita API"]
    P6 --> P62["6.2 Grabar Snapshot JSONB en Bitácora"]

    P7 --> P71["7.1 Generar Reportes R-01 a R-04"]
    P7 --> P72["7.2 Desplegar Dashboard de KPIs"]
    P7 --> P73["7.3 Purgar Adjuntos R2 (90–180 días)"]

    classDef rootNode fill:#0c4a6e,stroke:#082f49,stroke-width:2px,color:#ffffff;
    classDef mainProc fill:#e0f2fe,stroke:#0284c7,stroke-width:2px,color:#0f172a;
    classDef subProc fill:#f8fafc,stroke:#64748b,stroke-width:1px,color:#1e293b;
    class ROOT rootNode;
    class P1,P2,P3,P4,P5,P6,P7 mainProc;
    class P11,P12,P13,P21,P22,P23,P24,P31,P32,P41,P42,P51,P52,P53,P61,P62,P71,P72,P73 subProc;
```

## 1.4. Modelado de Eventos (Tabla de Eventos y Diagrama de Eventos)
Los eventos definen los estímulos externos, temporales o de estado que activan los procesos del sistema. Se incluyen los eventos operativos confirmados y los eventos de retención y sobrecargo definidos por la Dirección Técnica de ContaQuick PRO.
### 1.4.1 Tabla de Especificación de Eventos

# Tabla de Especificación de Eventos

| ID | Evento Disparador | Tipo de Evento | Agente / Origen | Proceso Activado | Respuesta / Salida Esperada |
|---|---|---|---|---|---|
| **EV-01** | Usuario solicita ingreso al módulo de soporte | Externo | AC-01, AC-02, AC-03 | 1.1, 1.2 (CU-01) | Sesión autenticada con JWT (usuarioId, rol, empresaId) y menú RBAC cargado. |
| **EV-02** | Cliente reporta incidencia con módulo ERP y capturas | Externo | AC-01 Cliente | 2.1, 2.2, 3.1, 3.2 (CU-02, CU-04) | Ticket creado en Abierto, adjuntos guardados en Cloudflare R2, técnico asignado y aviso en campanita. |
| **EV-03** | Usuario consulta progreso o historial de solicitudes | Externo | AC-01, AC-02 | 2.3 (CU-03) | Vista filtrada por tenant/rol con estados, adjuntos y línea de tiempo. |
| **EV-04** | Gerencia solicita reasignación manual de un caso | Externo | AC-03 Gerencia | 3.2, 6.1, 6.2 (CU-04) | Cambio de usuario_asignado_id, traza en historial y alerta en campanita. |
| **EV-05** | Técnico inicia atención, clasifica tipo o detecta duplicado | Externo | AC-02 Técnico / Dev | 2.4, 6.1 (CU-05) | Transición a En progreso/En espera, registro de tipo_id y vínculo ticket_padre_id. |
| **EV-06** | Incidencia confirmada como Falla del sistema (Bug) | De Estado | Proceso 2.4 | 4.1 (CU-08) | Creación o enlace 1:N en tbl_tareas_desarrollo con estado Kanban 'Por hacer'. |
| **EV-07** | Desarrollador mueve tarjeta en tablero Kanban/Lista | Externo | AC-02 Técnico / Dev | 4.2, 6.2 (CU-09) | Actualización de estado_kanban; si llega a 'Completada', sella fecha_cierre y notifica. |
| **EV-08** | Técnico ejecuta cierre e ingresa horas_invertidas > 0 | Externo | AC-02 Técnico / Dev | 5.1, 5.2, 5.3, 6.1 (CU-06, CU-07) | Ticket en Cerrado, cálculo de tiempo de resolución (fecha_cierre - fecha_creacion) y aviso a cliente/hijos. |
| **EV-09** | Consumo mensual de empresa alcanza o supera su plan | De Estado | Proceso 5.3 | 6.1, 6.2 (CU-07) | Notificación prioritaria SOPORTE_EXCESO_PLAN en la campanita de AC-03 Gerencia. |
| **EV-10** | Gerencia o técnico consulta dashboard y reportes | Externo | AC-03, AC-02 | 7.1, 7.2 (CU-10) | Generación al vuelo de reportes R-01 a R-04 y KPIs operativos. |
| **EV-11** | Cumplimiento de 90–180 días desde el cierre del ticket | Temporal | Reloj del Sistema (Cron) | 7.3, 6.2 | Purga automática de archivos pesados en Cloudflare R2 conservando bitácora de texto. |

### 1.4.2 Diagrama de Eventos del Sistema
```mermaid
flowchart LR
    subgraph EXT ["EVENTOS EXTERNOS (USUARIOS)"]
        EV1["EV-01: Solicitud de Acceso"]
        EV2["EV-02: Nuevo Reporte de Ticket"]
        EV3["EV-03: Consulta de Seguimiento"]
        EV4["EV-04: Reasignación Manual"]
        EV5["EV-05: Triaje y Agrupación"]
        EV7["EV-07: Movimiento en Kanban"]
        EV8["EV-08: Cierre con Horas Manuales"]
        EV10["EV-10: Solicitud de Reportes"]
    end

    subgraph PROC ["PROCESOS PRINCIPALES ACTIVADOS"]
        P1(("1.0 Seguridad<br/>RBAC / JWT"))
        P2(("2.0 Gestión de<br/>Tickets"))
        P3(("3.0 Balanceo<br/>de Carga"))
        P4(("4.0 Tablero<br/>Kanban Dev"))
        P5(("5.0 Cierre y<br/>Control Plan"))
        P6(("6.0 Campanita<br/>y Auditoría"))
        P7(("7.0 Analítica<br/>y Retención"))
    end

    subgraph INT ["EVENTOS DE ESTADO Y TEMPORALES"]
        EV6{"EV-06: Tipo =<br/>Falla de Sistema"}
        EV9{"EV-09: Consumo >=<br/>Plan Mensual"}
        EV11(["⏰ EV-11: Ticket Cerrado<br/>> 90–180 días"])
    end

    EV1 --> P1
    EV2 --> P2
    P2 --> P3
    EV3 --> P2
    EV4 --> P3
    EV5 --> P2
    P2 --> EV6
    EV6 --> P4
    EV7 --> P4
    EV8 --> P5
    P5 --> EV9
    EV9 --> P6
    P2 -.-> P6
    P3 -.-> P6
    P5 -.-> P6
    EV10 --> P7
    EV11 --> P7
    P7 -.-> P6

    classDef evExt fill:#fefce8,stroke:#ca8a04,stroke-width:1.5px,color:#713f12;
    classDef evInt fill:#fef2f2,stroke:#dc2626,stroke-width:1.5px,color:#7f1d1d;
    classDef proc fill:#e0f2fe,stroke:#0284c7,stroke-width:2px,color:#0c4a6e,font-weight:bold;
    class EV1,EV2,EV3,EV4,EV5,EV7,EV8,EV10 evExt;
    class EV6,EV9,EV11 evInt;
    class P1,P2,P3,P4,P5,P6,P7 proc;
```