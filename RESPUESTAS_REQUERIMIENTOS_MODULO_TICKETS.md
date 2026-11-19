# Respuestas a Requerimientos Técnicos y Operativos — Módulo de Tickets / Helpdesk
**ContaQuick PRO — ERP en la Nube (Honduras)**  
**Fecha:** 19 de noviembre de 2026  
**Dirigido a:** Equipo Evaluador / Consultoría de Desarrollo  
**Remitente:** Dirección Técnica — ContaQuick PRO  

---

### 1. Stack Tecnológico Actual
* **Frontend:** React 18, TypeScript, Tailwind CSS, Vite, Radix UI.
* **Backend:** Node.js, Express, TypeScript.
* **Base de Datos:** PostgreSQL (arquitectura multi-tenant: base maestra global `control` + una base de datos aislada por cada empresa).
* **Compatibilidad:** Si el módulo de soporte/tickets se construye con **React + Node.js/TypeScript**, la integración y el mantenimiento con nuestro equipo será completamente nativa.

---

### 2. Autenticación y Registro en Campanita de Notificaciones
* **Autenticación:** Vía **API REST (HTTP/JSON)** con **tokens JWT (Bearer Tokens)**. El usuario se autentica contra nuestra API y recibe su JWT conteniendo `usuarioId`, `rol` y `empresaId`.
* **Notificaciones:** Contamos con el endpoint y la tabla `control.notificaciones`. Para registrar avisos en la campanita, basta con que el servicio de tickets consuma la API interna (`POST /api/v1/notificaciones`) enviando el título, mensaje, tipo y usuario destinatario.
* **Restricciones de red:** No manejamos lista blanca rígida de IPs para clientes. Para la comunicación de servidor a servidor (Machine-to-Machine) dentro del VPS, se puede trabajar por red local (`localhost`) o mediante un token de servicio (`API Secret Key`) en los encabezados HTTP.

---

### 3. Almacenamiento de Archivos y Capturas de Tickets
* **Preferencia:** Para no saturar el almacenamiento en disco del servidor principal ni degradar el rendimiento de los respaldos de base de datos, preferimos utilizar almacenamiento de objetos compatible con S3 como **Cloudflare R2** o **AWS S3**.
* *(Recomendación: Cloudflare R2 es especialmente costo-eficiente al ofrecer 10 GB gratuitos y costo cero por transferencia de descarga/egress).*

---

### 4. Alojamiento del Módulo de Tickets (VPS Hostinger)
* Si el módulo es una aplicación ligera desarrollada a la medida en **Node.js/React**, puede residir en el **mismo VPS actual** (gestionado con PM2 y servido a través del reverse proxy de Nginx) sin acarrear costos adicionales de infraestructura.
* Solo si la solución es una plataforma de terceros prefabricada y pesada (que exija servicios dedicados como Redis, Elasticsearch, Sidekiq o Ruby con alto consumo de memoria RAM), convendría presupuestar un VPS independiente por un costo aproximado de **$10 a $18 USD / mes** en Hostinger.

---

### 5. Dominio y Certificado SSL
* **No es necesario presupuestar compra de dominio ni SSL.**
* El sistema correrá bajo un subdominio de nuestro dominio actual (por ejemplo, `soporte.contaquickpro.com` o `ayuda.contaquickpro.com`). Los certificados SSL los gestionamos con **Let's Encrypt / Certbot** de forma automatizada, renovable y sin costo.

---

### 6. Respaldos y Monitoreo en el VPS
* **Respaldos de Base de Datos:** Ya disponemos de scripts automatizados (`backup-db.sh`) que respaldan periódicamente todas las bases de datos de PostgreSQL con una política de retención de 14 días.
* **Monitoreo:** El estado de los procesos se gestiona internamente mediante PM2. Si en su propuesta contemplan una herramienta externa de monitoreo de disponibilidad (*uptime*) con alertas ante caídas, con gusto es bienvenida en la propuesta.

---

### 7. Tarifa por Hora de Soporte Adicional (Cálculo de ROI)
* Estimamos una tarifa de **$25 a $35 USD / hora** (aproximadamente **L. 650 a L. 850 HNL / hora**) para soporte técnico especializado, capacitaciones o consultoría personalizada que excedan las horas contempladas en cada plan.

---

### 8. Horas Semanales Perdidas en Gestión Manual (Ahorro Operativo)
* Se estima una pérdida de entre **8 y 12 horas semanales** dedicadas a clasificar mensajes dispersos de WhatsApp, correos sin seguimiento, reasignar manualmente quién atiende cada incidente y dar seguimiento interno a los desarrolladores. La centralización en una mesa de ayuda formal apunta a reducir más del 60% de este tiempo improductivo.

---

### 9. Factibilidad Legal y Privacidad de Datos SAR
* Aunque la normativa tributaria hondureña exige resguardar registros fiscales formales por 5 años, para **capturas de pantalla y archivos adjuntos de soporte**, se define la siguiente política:
  * **Retención de adjuntos:** 90 a 180 días una vez resuelto y cerrado el ticket; concluido dicho período, purga automática de los archivos pesados del storage conservando únicamente la bitácora de texto.
  * **Confidencialidad:** El acceso a los adjuntos con datos fiscales estará restringido con permisos estrictos únicamente al cliente titular del ticket y al personal técnico asignado.

---

### 10. Calendario, Adopción y Fechas Críticas
* **Familiaridad del equipo:** El equipo de desarrollo ya trabaja bajo metodologías ágiles y tableros tipo Kanban/Jira, por lo que **no se requiere capacitación intensiva**; únicamente una sesión breve de inducción sobre el flujo de estados y transiciones de los tickets.
* **Fechas críticas donde NO conviene realizar pruebas mayores o despliegues:**
  * **Días 10 al 12 de cada mes:** Presentación y liquidación de declaraciones mensuales de ISV ante el SAR (pico de facturación y consultas de clientes).
  * **Días 14-15 y 29-30/31 de cada mes:** Cierres de quincena y fin de mes (procesamiento de nóminas y balances contables).
  * **Ventana ideal de despliegue:** Entre los días 18 y 25 de cada mes, o en horarios nocturnos / fines de semana.
