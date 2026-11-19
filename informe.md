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
(aqui va el diagrama)

## 1.3. Diagrama de Descomposición Funcional
El Diagrama de Descomposición Funcional desglosa jerárquicamente el Proceso 0 en los 7 procesos principales derivados de las macrofunciones (MF-01 a MF-07) y en sus 19 procesos elementales ejecutables.
(aqui va el diagrama)

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