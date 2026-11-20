# Sistema de Gestión de Tickets y Tareas — ContaQuick PRO

![Estado](https://img.shields.io/badge/Estado-Unidad%20III%20En%20Progreso-0284c7?style=flat-square)
![UNAH](https://img.shields.io/badge/UNAH-DIA--158%20Arquitectura%20de%20Software%20II-1e3a8a?style=flat-square)
![Frontend](https://img.shields.io/badge/Frontend-React%2018%20%7C%20TypeScript%20%7C%20Tailwind-38bdf8?style=flat-square)
![Backend](https://img.shields.io/badge/Backend-Node.js%20%7C%20Express%20%7C%20TS-22c55e?style=flat-square)
![Database](https://img.shields.io/badge/Database-PostgreSQL%2014%2B%20(4FN)-336791?style=flat-square)
![Storage](https://img.shields.io/badge/Storage-Cloudflare%20R2-f97316?style=flat-square)

Portafolio técnico, modelado lógico y propuesta arquitectónica para el **Sistema de Gestión de Tickets y Tareas (Helpdesk & Kanban)** diseñado a la medida para el ERP en la nube **ContaQuick PRO** (Honduras).

> **Contexto Académico:** Proyecto desarrollado para la asignatura **DIA-158 Arquitectura de Software II** de la Carrera de Informática Administrativa, Facultad de Ciencias Económicas, Administrativas y Contables — **Universidad Nacional Autónoma de Honduras (UNAH)**.

---

## Descripción del Problema y Solución

Actualmente, **ContaQuick PRO** atiende las solicitudes de soporte técnico de forma dispersa mediante correo electrónico y organiza las tareas de desarrollo sin una herramienta centralizada, lo que impide medir tiempos de respuesta, balancear la carga entre sus tres desarrolladores y auditar las horas mensuales de soporte incluidas en cada plan contratado (ej. 15 horas mensuales en el Plan Pyme).

Este proyecto diseña un módulo web responsive completamente integrado a la infraestructura existente del ERP para:
* **Centralizar el ciclo de vida de tickets** en 4 estados oficiales (`Abierto`, `En progreso`, `En espera`, `Cerrado`), clasificados según los **24 módulos oficiales de ContaQuick PRO** y tipificados en *Error de uso* vs. *Falla del sistema*.
* **Balancear automáticamente la carga laboral** asignando cada nuevo ticket al técnico con menor cantidad de casos activos (`estado != 'Cerrado'`).
* **Controlar las horas de soporte por plan** mediante dos métricas diferenciadas sin tablas redundantes de tiempo: **Horas invertidas** (ingreso manual del técnico al cierre) y **Tiempo de resolución** (cálculo automático `fecha_cierre - fecha_creacion`).
* **Vincular soporte con ingeniería** agrupando tickets duplicados (`ticket_padre_id`) y conectando fallas reales a un tablero **Kanban / Lista** (`tbl_tareas_desarrollo` en relación `1:N`).
* **Reutilizar el ecosistema nativo del cliente**, autenticando vía JWT (`ContaQuickDBMaster`) y registrando avisos en el Centro de Notificaciones ("Campanita") mediante la API interna `POST /api/v1/notificaciones`.

---

## Arquitectura y Stack Tecnológico Oficial

Alineado de forma 100% nativa con el stack confirmado por la Dirección Técnica de ContaQuick PRO:

| Capa | Tecnologías y Estándares | Propósito en el Sistema |
| :--- | :--- | :--- |
| **Presentación (Frontend)** | React 18, TypeScript, Tailwind CSS, Vite, Radix UI | Apartado de soporte responsive, panel técnico, tablero Kanban/Lista y dashboard de KPIs en `soporte.contaquickpro.com`. |
| **Servicios (Backend)** | Node.js, Express, TypeScript, API REST (HTTP/JSON) | Lógica de negocio, validación de **JWT (Bearer Tokens)**, matriz dinámica **RBAC** e integración M2M (`API Secret Key`). |
| **Persistencia (Base de Datos)** | PostgreSQL 14+ (Normalizado hasta 4FN) | Esquema `soporte` (14 tablas, `UUID`, `snake_case`, auditoría `JSONB` *append-only*) acoplado al esquema maestro de `ContaQuickDBMaster`. |
| **Almacenamiento de Objetos** | Cloudflare R2 (S3-compatible) | Almacenamiento de capturas y adjuntos (`tbl_ticket_adjuntos`) con política de retención y purga automática a los 90–180 días. |
| **Infraestructura y Operaciones** | VPS Hostinger (Nginx + PM2), Let's Encrypt / Certbot | Despliegue en el servidor actual sin costo adicional de infraestructura, SSL automatizado y respaldos diarios (`backup-db.sh`, 14 días). |

---

## Mapa de Avance del Proyecto (Blueprint Levels 1–3)

### Unidad I — Definición, Alcance y Requerimientos (Completada)
* [x] Definición del problema, entrevista técnica y revisión documental.
* [x] Análisis de procesos *AS-IS* y *TO-BE* con puntos críticos.
* [x] Libro de Necesidades (`N-01` a `N-10`), Alcance Base (MVP) y Supuestos.
* [x] Libro de Costo inicial basado en horas hombre (512 h — L 141,280.00 / US$ 5,252.04).
* [x] Requerimientos Funcionales y No Funcionales (IEEE 830) y Casos de Prueba de Aceptación (BDD).

### Unidad II — Modelado Lógico y de Datos (Completada)
* [x] Catálogo oficial de 4 Actores (`AC-01` a `AC-04`) y 10 Casos de Uso (`CU-01` a `CU-10`).
* [x] Diagrama General de Casos de Uso, Narraciones detalladas, Jerarquización y Diagrama UML BPMN.
* [x] Matriz de Dependencias, Diagrama de Contexto y 7 Macrofunciones (`MF-01` a `MF-07`).
* [x] Modelos de Datos Iniciales, Jerarquías Generalizadas (rol solapado `AC-02`) y 4 Reportes Gerenciales (`R-01` a `R-04`).
* [x] Demostración de Normalización (`0FN` → `1FN` → `2FN` → `3FN` → `BCNF/4FN`), Subsistema RBAC (6 tablas), DER Definitivo y Diccionario de Datos Físico.

### Unidad III — Diseño Lógico Final, Factibilidad y Propuesta (En Progreso)
* [x] Levantamiento técnico final con ContaQuick PRO (Stack, API JWT, Campanita, R2, VPS, ROI y ventanas de despliegue).
* [x] DFD de Contexto (Nivel 0), Diagrama de Descomposición Funcional y Modelado de Eventos (`EV-01` a `EV-11`).
* [ ] DFDs Elementales por proceso (`P1` a `P7`) integrados en el informe.
* [ ] Matriz de Sincronización, Diagrama de Componentes UML y Diagrama de Arquitectura (3 Capas).
* [ ] Estudio Integral de Factibilidad (Técnica, Operativa, Cultural/Política, Calendario, Económica y Legal).
* [ ] Análisis de Costo-Beneficio (CAPEX + OPEX $0/mes, ahorro operativo y ROI).
* [ ] Propuesta del Sistema (Matrices de Decisión + Infografía Comercial) y Defensa Final.

---

## Aspectos Destacados del Diseño de Datos

* **Seguridad Dinámica RBAC y Auditoría Forense:** Implementa `tbl_usuarios`, `tbl_roles`, `tbl_usuarios_roles` (relación M:N para el rol solapado de los 3 desarrolladores), `tbl_objetos`, `tbl_permisos` y `tbl_bitacora_auditoria` con snapshots `JSONB`.
* **Regla de Cero Tabla de Horas:** Las métricas de tiempo se calculan directamente desde `tbl_tickets` (`horas_invertidas`, `fecha_creacion`, `fecha_cierre`) contrastadas contra `horas_mensuales_plan` de `global_empresas`.
* **Privacidad y Cumplimiento SAR:** Los adjuntos fiscales residen en Cloudflare R2 con acceso restringido al titular y técnico asignado, y purga automática entre 90 y 180 días después del cierre.

---

## Equipo del Proyecto

| No. | Integrante | Número de Cuenta | Rol Principal en el Proyecto |
| :-: | :--- | :---: | :--- |
| 1 | **Néstor Julián Déctor Rojas** | `20232000391` | Desarrollador Front-end y Especialista QA |
| 2 | **Norma Xiomara Montoya Rivera** | `20211001882` | Desarrollador Senior y Enlace con ContaQuick PRO |
| 3 | **Gensy Andony Díaz Aguilar** | `20221000528` | Desarrollador Senior y Desarrollador Front-end |
| 4 | **Anthony Moisés Barahona Rodríguez** | `20231001719` | Arquitecto de Software y Desarrollador Senior |
| 5 | **Angie Yisel Cerrato Flores** | `20231002845` | Analista de Sistemas, Diseñador UX/UI y Líder de Proyecto |

---

## Catálogo de Documentación y Diagramas

### Documentación del Cliente e Informes
* [Respuestas Técnicas y Operativas — ContaQuick PRO (19/11/2026)](docs/cliente/respuestas-requerimientos-modulo-tickets.md)
* [Informe de Avance — Unidad II (Modelado Lógico y de Datos)](docs/unidad-2/informe-unidad-2.md)
* [Informe de Avance — Unidad III (Diseño Lógico, Análisis de Decisión y Propuesta del Sistema)](docs/unidad-3/informe-unidad-3.md)

### Diagramas — Unidad II (Modelado Lógico y de Datos)
| # | Artefacto | Archivo |
| :-: | :--- | :--- |
| 01 | Diagrama General de Casos de Uso (UML) | [Ver SVG](docs/unidad-2/diagramas/01-casos-de-uso.svg) |
| 02 | Diagrama de Proceso de Negocio (UML BPMN) | [Ver SVG](docs/unidad-2/diagramas/02-bpmn-procesos.svg) |
| 03 | Diagrama de Contexto del Sistema | [Ver SVG](docs/unidad-2/diagramas/03-contexto-sistema.svg) |
| 04 | Diagrama de Macrofunciones (`MF-01` a `MF-07`) | [Ver SVG](docs/unidad-2/diagramas/04-macrofunciones.svg) |
| 05 | Modelo de Datos de Contexto | [Ver SVG](docs/unidad-2/diagramas/05-modelo-datos-contexto.svg) |
| 06 | Modelo de Datos Basado en Claves (*Key-Based*) | [Ver SVG](docs/unidad-2/diagramas/06-modelo-datos-basado-en-claves.svg) |
| 07 | Jerarquías Generalizadas (Rol Solapado `AC-02`) | [Ver SVG](docs/unidad-2/diagramas/07-jerarquias-generalizadas.svg) |
| 08 | Modelo Entidad-Relación Definitivo (DER 4FN) | [Ver SVG](docs/unidad-2/diagramas/08-modelo-entidad-relacion.svg) |

### Diagramas — Unidad III (Modelado de Procesos y Arquitectura)
| # | Artefacto | Archivo |
| :-: | :--- | :--- |
| 01 | DFD de Contexto (Nivel 0) | [Ver SVG](docs/unidad-3/diagramas/01-dfd-contexto-nivel-0.svg) |
| 02 | Diagrama de Descomposición Funcional | [Ver SVG](docs/unidad-3/diagramas/02-descomposicion-funcional.svg) |
| 03 | Diagrama de Eventos (`EV-01` a `EV-11`) | [Ver SVG](docs/unidad-3/diagramas/03-diagrama-eventos.svg) |
| 04 | DFD Elemental `P1`: Seguridad JWT y RBAC | [Ver SVG](docs/unidad-3/diagramas/04-dfd-p1-seguridad-rbac.svg) |
| 05 | DFD Elemental `P2` y `P3`: Tickets, Cloudflare R2 y Balanceo | [Ver SVG](docs/unidad-3/diagramas/05-dfd-p2-p3-tickets-balanceo.svg) |
| 06 | DFD Elemental `P2` y `P4`: Triaje, Padre-Hijo y Kanban | [Ver SVG](docs/unidad-3/diagramas/06-dfd-p2-p4-triaje-kanban.svg) |
| 07 | DFD Elemental `P5` y `P6`: Cierre, Horas, Plan y Campanita | [Ver SVG](docs/unidad-3/diagramas/07-dfd-p5-p6-cierre-horas-campanita.svg) |
| 08 | DFD Elemental `P7`: Analítica (`R-01` a `R-04`) y Retención R2 | [Ver SVG](docs/unidad-3/diagramas/08-dfd-p7-analitica-retencion.svg) |



## Licencia y Uso

Documentación técnica y arquitectónica elaborada con fines académicos y de vinculación universidad-empresa para **ContaQuick PRO** (UNAH, 2026).