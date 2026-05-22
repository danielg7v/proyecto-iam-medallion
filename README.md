# proyecto-iam-medallion
Pipeline de auditoría de identidades y accesos (IAM) utilizando Arquitectura Medallion en Databricks, automatizado con Jobs y consultado mediante Databricks Genie.

# Auditoría Continua de Identidades y Accesos (IAM) mediante Arquitectura Medallion

## Información del Proyecto
* **Institución:** UNAULA
* **Materia:** Big Data
* **Docente:** Yeis
* **Integrantes:**
  * Daniel Gonzalez
  * Jose Santiago Valencia
  * Sebastian Castrillon

---

## Descripción del Proyecto
Este proyecto implementa un pipeline de Big Data optimizado para la gestión y auditoría automatizada de Identidades y Accesos (IAM - Identity and Access Management). Utilizando el entorno corporativo de **Databricks** y el motor de procesamiento distribuido **Apache Spark**, el sistema es capaz de contrastar la "Realidad" de los privilegios asignados en aplicaciones críticas (**Jira y Appgate**) frente al "Deber Ser" corporativo dictado por una **Matriz de Modelamiento de Roles**.

Para simular un entorno empresarial real, el pipeline se ejecuta de forma automatizada mediante un **Scheduled Job cada 8 días**, lo que garantiza un monitoreo constante y la detección temprana de brechas de seguridad informáticas (como cuentas huérfanas o privilegios excesivos).

---

## Arquitectura del Dato: Modelo Medallion

El procesamiento de datos se diseñó bajo el patrón de arquitectura **Medallion (Bronze, Silver, Gold)** para garantizar la limpieza progresiva y el gobierno del dato:

```text
  [ Fuentes Crudas ] ──>  🥉 Capa Bronze  ──>  🥈 Capa Silver  ──>  🥇 Capa Gold
  (IdentityX, Plantas,     (Delta Tables        (Resolución de       (Motor de Auditoría
   Extractos de Apps)       Inmutables)          Identidades)         & KPIs de Control)

🥉 1. Capa Bronze (Ingesta y Persistencia)
En esta fase se realiza la ingesta cruda de las fuentes de datos. Para el proyecto se han modelado e insertado datos sintéticos robustos (libres de caracteres especiales, tildes y con restricción de usuarios de red alfanuméricos únicos) que reflejan fielmente las estructuras de una organización:

identityx: El maestro autorizado de recursos humanos (2,000 empleados únicos).

planta_1 (Konecta) & planta_2 (Teleperformance): Diccionarios de control para personal externo (BPO), actuando como puente para traducir correos electrónicos del proveedor a usuarios de red del banco.

matriz_modelamiento: El estándar de gobierno de seguridad que mapea los Roles de Negocio con los Roles Técnicos aprobados.

gda_appgate_usuarios_grupos & jira_usuarios_permisos: Extractos reales de uso de las aplicaciones (con una inyección controlada del 10% de anomalías de seguridad para evaluar el motor de auditoría).

🥈 2. Capa Silver (Limpieza e Integración)
El objetivo primordial de la capa Silver es la Resolución de Identidades. Los extractos de aplicaciones usan llaves distintas (Jira usa correo electrónico y Appgate usa usuario de red).
Mediante transformaciones avanzadas en Spark SQL y funciones lógicas (COALESCE), se unificaron de forma vertical todos los accesos. Se cruzaron los correos corporativos externos con las Plantas BPO para obtener el usuario raíz, y posteriormente se enriqueció la información haciendo un LEFT JOIN hacia identityx.

Resultado: La tabla única proyecto_iam_silver.maestro_accesos.

🥇 3. Capa Gold (Motor de Cumplimiento y Negocio)
Es la capa de valor de negocio y auditoría. Aquí la data integrada se contrasta de forma estricta contra la matriz_modelamiento. El pipeline ejecuta un análisis lógico condicional (CASE WHEN) para clasificar de manera automática cada acceso bajo uno de los siguientes estados:

Modelado Correcto: El usuario cuenta exactamente con el permiso aprobado para su cargo.

Anomalía (Acceso Incorrecto): El cargo está modelado, pero el usuario posee un rol técnico no autorizado (Alarma de seguridad).

Cuenta Huérfana: El usuario tiene acceso activo en Jira o Appgate, pero ya no existe en el maestro de recursos humanos identityx (Alto riesgo de intrusión).

No Modelado: El usuario existe pero su cargo aún no cuenta con un estándar de gobierno definido.

Resultado: La tabla refinada proyecto_iam_gold.reporte_auditoria.

⚙️ Automatización y Operación (Orchestration)
Para asegurar que el control de accesos no dependa de ejecuciones manuales, el notebook completo fue empaquetado y programado como un Databricks Scheduled Job:

Frecuencia: Cada 8 días.

Propósito: Re-procesar las bases de datos de forma periódica para detectar bajas de personal (Cuentas Huérfanas creadas en los últimos 7 días) o cambios no autorizados en los aplicativos.
