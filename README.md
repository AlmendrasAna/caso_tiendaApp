# caso_tiendaApp

-----


## Componentes Críticos y Posibles Fallas en TiendApp

- Dado el contexto de TiendApp y las quejas de los clientes, es crucial establecer un monitoreo integral en varios niveles de su infraestructura.

Aquí están los componentes críticos que deberían estar siendo monitoreados:

| Componente del Sistema | Tipo de Falla / Problema Potencial | Impacto en TiendApp |
|------------------------|------------------------------------|---------------------|
| **Backend (Node.js en EC2)** | **CPU alta:** Aplicación saturada, bucles infinitos, consultas ineficientes. | Demoras en la respuesta de la aplicación, lentitud general. |
|                        | **Memoria insuficiente:** `Out of Memory` (OOM), cuelgues del proceso Node.js. | Caídas del sistema, errores 500, no se cargan productos. |
|                        | **Errores 500 en API (internos del servidor):** Fallos en el código, excepciones no manejadas, lógica de negocio defectuosa. | Transacciones fallidas (pagos, carga de productos), quejas de clientes. |
|                        | **Latencia alta en respuestas API:** Retraso en el procesamiento de solicitudes. | Lentitud en la navegación y operación para el usuario. |
|                        | **Fugas de memoria:** Consumo gradual e incontrolado de memoria. | Degradación del rendimiento a lo largo del tiempo, eventuales caídas. |
|                        | **Errores de conexión a servicios externos:** Fallo al comunicarse con pasarelas de pago o APIs de terceros. | Demoras en pagos, fallos en la confirmación de pedidos. |
| **Base de Datos (PostgreSQL en RDS)** | **Conexiones máximas alcanzadas:** No se pueden establecer nuevas conexiones a la base de datos. | Aplicación no puede acceder a los datos, errores de servicio. |
|                        | **Consultas SQL lentas:** Queries no optimizadas, falta de índices. | Demoras en la carga de productos, procesamiento de pedidos lento, pagos lentos. |
|                        | **Bloqueos de base de datos:** Transacciones prolongadas que bloquean otras operaciones. | Datos inconsistentes, operaciones que no finalizan. |
|                        | **IOPS saturadas:** Disco de la base de datos no puede manejar la carga de lectura/escritura. | Degradación severa del rendimiento de la base de datos. |
|                        | **Espacio de almacenamiento insuficiente:** Base de datos se llena y no puede escribir nuevos datos. | Fallos al crear nuevos registros (pedidos, usuarios). |
| **Frontend (React en S3 + CloudFront)** | **Errores 4xx (Cliente):** Archivos no encontrados (404), problemas de permisos (403). | La página web no carga correctamente, recursos faltantes (imágenes, CSS, JS). |
|                        | **Errores 5xx (Servidor de CloudFront/S3):** Problemas en la distribución de la CDN o en el bucket S3. | El sitio web completo o partes de él no son accesibles para el usuario. |
|                        | **Lentitud en la carga inicial de la página:** Gran tamaño de los activos, configuración deficiente de CloudFront. | Mala experiencia de usuario, usuarios abandonan el sitio. |
|                        | **Errores de JavaScript en el navegador:** Fallos en la lógica del frontend. | Funcionalidad de la interfaz de usuario no trabaja (ej. botones no funcionan, formularios no envían). |
| **Despliegue (GitHub Actions)** | **Fallo en el pipeline de despliegue:** Errores en la configuración, dependencias faltantes, pruebas fallidas. | Nuevas versiones del software no se despliegan, errores en producción persisten. |
|                        | **Despliegues lentos:** Pipeline con demasiados pasos o pruebas ineficientes. | Retraso en la entrega de nuevas características o correcciones de errores. |
|                        | **Rollbacks fallidos:** Problemas al revertir a una versión anterior. | Mayor tiempo de inactividad durante una interrupción. |

---
## 📊 Métricas Esenciales para el Monitoreo en TiendApp

## 📊 Métricas Esenciales para Detectar Problemas en TiendApp

| Métrica 📊 | ¿Qué problema ayuda a detectar? 🔍 | ¿Por qué es crítica? ⚠️ |
|------------|------------------------------------|--------------------------|
| **⏱️ Tiempo de respuesta del API principal** | Lentitud en carga de productos y pagos | Un aumento constante puede señalar cuellos de botella en el backend (Node.js), afectando la experiencia del usuario. |
| **🚨 Número de errores HTTP 5xx en endpoints críticos** | Caídas del sistema, fallas en pagos o carga de productos | Estos errores indican que el servidor no está manejando correctamente las solicitudes. Son síntomas de problemas serios en la aplicación. |
| **🧠 Uso de CPU y RAM en instancias EC2** | Inestabilidad y caídas del sistema | El uso excesivo de recursos puede causar reinicios o fallos en el backend. Es clave para prever saturación del servidor. |
| **📉 Tasa de abandono en procesos de compra** | Interrupciones o errores en pagos | Una baja tasa de conversión o abandonos repentinos pueden revelar errores en el flujo de pago o lentitud que frustra al usuario. |
| **🗄️ Latencia y caídas en la base de datos (RDS)** | Fallos al cargar productos o procesar pedidos | Una base de datos lenta o sobrecargada puede provocar bloqueos en el sistema y demoras visibles para el usuario final. |

## Propuesta de Monitoreo para TiendApp con Prometheus y Grafana

Herramientas a Utilizar: Prometheus y Grafana

- Prometheus: Actuará como el motor principal de recolección y almacenamiento de métricas. Utiliza un modelo de extracción (pull) donde consulta endpoints HTTP de las aplicaciones o de exportadores (exporters) para obtener las métricas. Es ideal para monitorear el estado de los servicios y recursos a lo largo del tiempo.

- Grafana: Será la plataforma para la visualización de datos, la creación de dashboards interactivos y la configuración de alertas. Se integrará con Prometheus para consultar las métricas almacenadas y presentarlas de manera comprensible y accionable.

Las alertas deben ser configuradas en Grafana utilizando Prometheus como fuente de datos. El canal de notificación principal será Slack, que ya es comúnmente utilizado por equipos de desarrollo.

## 🔌 Integraciones Necesarias (Prometheus + Grafana)

| Componente | Integración Técnica |
|------------|----------------------|
| **Node.js (Backend en EC2)** | Exponer métricas con `express-prometheus-middleware` en `/metrics`. Prometheus scrapea este endpoint. |
| **PostgreSQL (RDS)** | Usar `postgres_exporter` en una instancia EC2. Requiere usuario de solo lectura en la base de datos. |
| **Instancia EC2 (Backend)** | Instalar `node_exporter` para obtener métricas del sistema: CPU, RAM, disco, red. |
| **Frontend (React en S3 + CloudFront)** | No se conecta directamente. Métricas indirectas pueden exponerse desde el backend sobre actividad de usuarios. |
| **Slack (Notificaciones)** | Configurar Alertmanager con webhook hacia un canal de Slack (`#dev-alertas`) para recibir notificaciones. |

## 📈 Dashboards Principales en Grafana

| Dashboard | Métricas Incluidas |
|-----------|--------------------|
| **🧠 Recursos del Servidor (EC2 con Node Exporter)** | - CPU (uso, load average)<br>- RAM utilizada vs disponible<br>- Disco (espacio, IOPS)<br>- Red (ingreso/egreso) |
| **⚙️ Backend Node.js (API)** | - Latencia de endpoints<br>- Tasa de peticiones<br>- Errores HTTP 5xx/4xx<br>- Tiempo promedio por endpoint |
| **🗄️ Base de Datos PostgreSQL** | - Conexiones activas<br>- Latencia de queries<br>- Uso de disco<br>- Locks y errores |
| **💳 Transacciones y Flujo de Compras (custom)** | - Tasa de éxito de pagos (expuesta desde backend)<br>- Errores en pagos<br>- Tiempo promedio de procesamiento |
## 🚨 Simulación de Alertas y Respuesta

### 🔔 Alertas a Configurar

| Alerta | Umbral sugerido | Frecuencia de chequeo |
|--------|------------------|------------------------|
| ⏱️ Latencia alta en API | > 500 ms (p95) durante 5 min | Cada 1 min |
| 🚨 Errores HTTP 5xx > 1% | Por más de 5 min consecutivos | Cada 1 min |
| 💾 CPU EC2 > 80% | Durante 10 minutos | Cada 2 min |
| 🧠 RAM EC2 > 85% | Uso sostenido por 10 minutos | Cada 2 min |
| 🗄️ Conexiones activas en RDS > 90% | Por 5 min continuos | Cada 1 min |
| 📉 Éxito en pagos < 95% | Durante cualquier período de 5 min | Cada 1 min |

---

### ⏰ Protocolo de Respuesta Fuera del Horario Laboral

1. **Alerta se dispara en Prometheus → Alertmanager** la envía a Slack.
2. **Notificación automática** etiqueta al equipo responsable y deja logs adjuntos.
3. Si es una alerta **crítica** (API caída, CPU al 100%, pagos fallando):
   - Se activa una **política de on-call** (turnos rotativos fuera de horario).
   - Se notifica vía Slack + Email .
4. El **responsable on-call**:
   - Consulta el dashboard en Grafana.
   - Verifica logs.
   - Toma acción pertinentes(reinicio, rollback, escalado manual o automático).

---

### ⚙️ Acciones Automatizadas Recomendadas

| Acción | Cuándo aplicarla | Cómo implementarla |
|--------|------------------|---------------------|
| 🔄 Reinicio automático del backend (Node.js) | Si error 5xx persiste mayor a 5 min | Script Bash con `systemctl restart` o llamado desde webhook. |
| 📈 Autoescalado de instancias EC2 | CPU > 80% por más de 10 min | Usar políticas de escalado en Auto Scaling Groups. |
| 🧼 Limpieza de logs o caché | Si uso de disco > 90% | Cron job que borra logs antiguos/locales. |
| 🚧 Modo mantenimiento | Si pagos caen a < 90% éxito | Backend responde con banner a usuarios hasta resolver. |

## ✅ Evaluación y Continuidad del Sistema de Monitoreo

### 📊 ¿Cómo se evaluará el éxito del sistema de monitoreo?

El éxito del sistema de monitoreo se evaluará con base en:

- Reducción del tiempo promedio de detección de incidentes (MTTD).
- Mejora en el tiempo de resolución (MTTR).
- Menor dependencia de reportes de clientes para detectar problemas.
- Aumento en la estabilidad del sistema (menos caídas no detectadas).
- Calidad y utilidad de los dashboards y alertas.

---

### 📈 KPIs para Medir la Mejora

| KPI | Objetivo |
|-----|----------|
| 🕒 MTTD (Mean Time to Detect) | mayor a 2 minutos desde que ocurre un problema. |
| 🛠️ MTTR (Mean Time to Recovery) | mayor a 15 minutos en horas laborales. |
| ⚠️ Número de incidentes detectados por usuarios | Reducción progresiva mensual. |
| 📉 Errores 5xx por cada 10.000 requests | Mantener < 5 errores. |
| ✅ Tasa de éxito en transacciones de pago | Mantener ≥ 98% mensualmente. |
| 📥 Alertas falsas positivas |mayor a 5% del total mensual de alertas. |

---

### 🔄 Integración con el Flujo DevOps de TiendApp

El monitoreo se integrará en el ciclo DevOps de TiendApp mediante:

- 📦 **Integración continua (CI/CD):**
  - Verificación automática de endpoints y métricas en cada despliegue.
  - Validación de que las métricas custom siguen siendo expuestas (health checks).
  
- 🔁 **Ciclo de feedback continuo:**
  - Cada incidente registrado en alertas se revisará en post-mortem semanales.
  - Los insights extraídos se integrarán como mejoras en infraestructura o código.
  
- 🛠️ **Dashboards como parte del flujo de trabajo diario:**
  - Grafana estará visible en pantallas de monitoreo o integrado en herramientas de comunicación (Slack, Notion, etc.).
  
- 🚀 **Alertas en pre-producción y staging:**
  - Monitoreo también activo en entornos previos a producción para detección anticipada de errores.

