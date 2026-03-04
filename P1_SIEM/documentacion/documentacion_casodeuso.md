# Caso de uso: Detección de fuerza bruta SSH e ICMP

**Autor:** Abel García Domínguez  
**Proyecto:** SIEM — Detección de incidentes de seguridad  
**Fecha:** Febrero 2026

---

## Descripción

Caso de uso diseñado para detectar y correlacionar **ataques de fuerza bruta SSH** y **escaneos ICMP** en un entorno SIEM, utilizando reglas de umbral basadas en volumen de eventos desde una misma IP origen.

---

## Objetivo

Detectar actividades maliciosas en la red monitorizada, correlacionando múltiples eventos desde un mismo origen para generar alertas cuando se superen umbrales específicos de fuerza bruta SSH y escaneos ICMP.

---

## Alcance

### Servicios monitorizados

- Servidores con SSH accesible (`TCP/22`)
- Hosts que responden a `ICMP echo-request` / `echo-reply`

### Tipos de actividad detectados

| Actividad | Umbral |
|-----------|--------|
| Intentos SSH fallidos desde misma IP | ≥ 5 en 5 minutos |
| Paquetes ICMP desde misma IP (ping sweep) | ≥ 50 en 60 segundos |

> **Excluye:** Fallos aislados y tráfico ICMP legítimo de monitorización puntual.

---

## Fuentes de eventos

| Plataforma | Tipo de log |
|------------|-------------|
| IDS Snort | Alertas SSH brute force y ICMP scan/flood |

### Campos clave en eventos

| Campo | Descripción |
|-------|-------------|
| `@timestamp` | Fecha y hora del evento |
| `src_ip` | IP origen |
| `dest_ip` | IP destino |
| `alert.signature` | Tipo de alerta |
| `proto` | Protocolo (TCP / UDP / ICMP) |
| `dest_port` | Puerto destino (ej. `22` para SSH) |

---

## Flujo lógico de detección

```
Snort analiza tráfico
        │
        │  genera alertas SSH / ICMP
        ▼
    Filebeat
        │
        │  envía logs → Logstash (puerto 5044)
        ▼
  Elasticsearch
        │
        │  indexa bajo ids-*
        ▼
     Kibana
        │
        └─ correlaciona eventos y aplica umbrales
```

### Umbrales configurados

| Tipo de ataque | Umbral |
|----------------|--------|
| SSH Fuerza Bruta | ≥ 5 alertas / 5 min |
| ICMP Escaneo | ≥ 50 eventos / 60 seg |

### La alerta generada incluye

- IP origen y destino
- Tipo de ataque
- Número de eventos
- Intervalo temporal

---

## Visualización y notificación

### Dashboard Kibana

- Panel dedicado **"Fuerza Bruta SSH/ICMP"**
- Filtros por `src_ip`, severidad e intervalo temporal

### Formato de alerta (entorno real)

```
Fecha:         [timestamp]
IP Origen:     [src_ip]
Tipo:          [SSH Brute Force / ICMP Scan]
Eventos:       [N] en [X] minutos
Enlace Kibana: [dashboard]
```

**Destinatarios:** `soc@empresa.local`, equipo de seguridad

---

## Matriz de severidad

| Tipo de alerta | Severidad |
|----------------|-----------|
| SSH Brute Force (IP externa) | Alta |
| SSH Brute Force (IP interna) | Crítica |
| ICMP Scan múltiple hosts | Media |

---

## Medidas de respuesta recomendadas

### Endurecimiento inmediato

- **SSH:** Autenticación por claves públicas, listas blancas de IP, cambio del puerto `22`
- **Fail2ban:** Bloqueo automático de IPs maliciosas
- **Firewall:** Reglas dinámicas basadas en alertas SIEM

---

## Mantenimiento del caso de uso

- Revisar umbrales según la tasa de falsos positivos observada
- Añadir excepciones para tráfico de monitorización legítima
- Correlacionar con otros casos de uso (malware, port scanning, etc.)

---

## Buenas prácticas

- Deshabilitar servicios inseguros (Telnet, RSH)
- Mantener actualizados IDS, SIEM y sistemas operativos
- Integrar múltiples casos de uso para detectar ataques compuestos