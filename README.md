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
