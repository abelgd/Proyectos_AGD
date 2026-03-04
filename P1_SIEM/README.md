# Detección de Incidentes IDS/IPS/SIEM

**Autor:** Abel García Domínguez  
**Proyecto:** Implementación de un entorno SIEM para detección de incidentes  
**Tecnologías:** ELK Stack (Elasticsearch, Logstash, Kibana), Filebeat, Nginx, Snort, Docker, Hydra

---

## Descripción del proyecto

Este proyecto presenta la implementación y configuración de un entorno **SIEM** (*Security Information and Event Management*) basado en la **ELK Stack**, junto con un agente de recolección de logs y la integración de un **IDS** (Snort) para detección de amenazas en red.

El objetivo es diseñar y desplegar un entorno funcional de monitorización y correlación de eventos capaz de detectar un **ataque de fuerza bruta SSH**, analizando el flujo completo desde la generación del log hasta su visualización en Kibana.

---

## Estructura del proyecto

El repositorio se compone de tres bloques principales de documentación técnica:

### 1. Instalación y configuración del entorno
> **Archivo:** **[Instalación y configuración](documentacion/instalacion_configuracion.md)**

Contiene la definición de la arquitectura (contenedores y red Docker), despliegue de ELK (v7.16.3), creación del agente Nginx–Filebeat, configuración de Filebeat, validación de ingesta en Kibana y resolución de problemas comunes (TLS, versiones o procesos detenidos).

### 2. Caso de uso: Detección de fuerza bruta SSH
> **Archivo:** **[Documentación de caso de uso](documentacion/documentacion_casodeuso.md)**

Describe los objetivos de detección, los activos involucrados, escenarios de ataque, fuentes de logs, reglas IDS utilizadas y criterios de verificación de la alerta de fuerza bruta.

### 3. Implementación práctica del caso de uso
> **Archivo:** **[Implementación de caso de uso](documentacion/implementacion_casodeuso.md)**

Incluye la instalación y configuración de Snort 2.9.20, pruebas de validación (ICMP, SSH), ejecución de un ataque de fuerza bruta con Hydra, análisis de alertas y ajuste final de reglas y salida de Snort para la ingesta por Filebeat.

---

## Arquitectura técnica

### Componentes principales

| Rol | Componente | Detalle |
|-----|-----------|---------|
| **SIEM / SOC** | Contenedor `sebp/elk:7.16.3` | Kibana → puerto `5601` · Elasticsearch → puerto `9200` · Logstash (input Beats) → puerto `5044` |
| **Agente / Endpoint** | Contenedor `nginx-filebeat` | Nginx (generación de logs) → puerto `8080` · Filebeat (envío de logs a Logstash) |
| **Atacante** | Máquina Ubuntu (WSL) | Ejecución de Hydra para simular ataques SSH |

### Flujo de datos

```
Nginx / Snort
     │
     │  generan logs en:
     │  /var/log/nginx/*.log
     │  /var/log/snort/alert
     ▼
  Filebeat
     │
     │  monitoriza y envía eventos a Logstash (puerto 5044)
     ▼
  Logstash
     │
     │  procesa y reenvía a Elasticsearch
     ▼
Elasticsearch
     │
     │  indexa bajo el patrón filebeat-*
     ▼
  Kibana
     │
     └─ visualización en el módulo Discover
```

---

## Casos de uso y validación

El caso principal reproducido es un escenario de **fuerza bruta SSH**, detectado mediante alertas generadas por Snort e indexadas en Kibana.

Durante la validación se identificaron y resolvieron incidencias relacionadas con:

- El **parsing de logs** y el uso de reglas dependientes del flujo establecido (`flow:established`).
- La solución final consistió en **ajustar la regla IDS** y configurar la salida de Snort a fichero para su posterior ingesta mediante Filebeat.

---

## Evidencias

Las capturas, gráficos y validaciones visuales del entorno y del caso de uso se encuentran en la carpeta:

```
/documentacion/img
```

---

## Resultados

Este proyecto demuestra la capacidad de integrar herramientas de **detección** (IDS), **gestión de eventos** (SIEM) y **recolección de logs** (Filebeat) en un entorno completamente funcional, mostrando cómo se detecta, procesa y visualiza un incidente de seguridad real.

Representa una práctica completa de **monitorización, correlación y análisis forense básico**, como base de un entorno SOC educativo o de laboratorio.