# Implementación práctica: Detección fuerza bruta SSH/ICMP

**Autor:** Abel García Domínguez  
**Entorno:** Laboratorio Docker (ELK 7.16.3 + Nginx/Filebeat + Snort 2.9.20)  
**Fecha:** Febrero 2026

---

## Objetivo

Desplegar el flujo completo de detección desde **Snort → Filebeat → ELK**, validando:

1. Generación de alertas ICMP/SSH por Snort
2. Ingesta automática por Filebeat
3. Indexación y visualización en Kibana
4. PoC real con ataque Hydra (fuerza bruta SSH)

---

## Entorno base

```
SIEM: sebp/elk:7.16.3
├── Kibana:              5601
├── Elasticsearch:       9200
└── Logstash (Beats):    5044

Agente: nginx-filebeat + Snort 2.9.20
├── Nginx:               8080 (logs web)
├── Snort:               eth0 → /var/log/snort/alert
└── Filebeat:            → Logstash 5044

Red: elk-red (Docker network)
```

---

## 1. Configuración Snort (IDS)

### Reglas implementadas (`local.rules`)

```snort
# ICMP - Validación básica
alert icmp any any -> any any (msg:"ICMP Traffic"; sid:1000001;)

# SSH - Fuerza bruta (ajustada)
alert tcp any any -> $HOME_NET 22 (msg:"SSH Brute Force Attempt"; \
    flow:to_server,not_established; threshold:type both, track by_src, count 5, seconds 300; \
    sid:1000002;)
```

### Incidencia resuelta: `flow:established`

| | Detalle |
|---|---------|
| **Problema** | La regla solo detectaba conexiones establecidas, no los intentos fallidos generados por Hydra |
| **Solución** | Modificada a `flow:to_server,not_established` + threshold por IP origen |

### Ejecución Snort (salida para Filebeat)

```bash
snort -A fast -q -c /etc/snort/snort.conf -i eth0 -k none -l /var/log/snort
```

| Flag | Descripción |
|------|-------------|
| `-A fast` | Formato legible para ingesta |
| `-l /var/log/snort` | Directorio monitoreado por Filebeat |
| `-k none` | Bypass de checksums (entorno virtualizado) |

---

## 2. Pipeline de ingesta (Filebeat → ELK)

### Filebeat config (`filebeat.yml`)

```yaml
filebeat.inputs:
  - type: log
    paths:
      - /var/log/snort/alert

output.logstash:
  hosts: ["elk:5044"]   # Resolución por nombre Docker
```

### Validación de conectividad

```bash
filebeat test config -e
filebeat test output -e
```

> Config OK — conexión Logstash activa.

---

## 3. Kibana: Indexación y Discover

### Patrón de índice

```
filebeat-*   (timestamp: @timestamp)
```

### Campos disponibles en eventos Snort

```
message:    "[1:1000002:1] SSH Brute Force Attempt [src_ip:192.168.x.x]"
src_ip      dest_ip      proto      dest_port
```

---

## 4. Pruebas de validación (PoC)

### Test 1 — ICMP (Ping sweep)

```bash
ping -c 50 <endpoint>
```

**Resultado:** Alertas Snort → Filebeat → Kibana (Discover muestra los nuevos eventos)

### Test 2 — Fuerza bruta SSH (Hydra)

```bash
# Preparación (scripts automatizados)
./crear_claves
./crear_usuarios

# Ataque
./lanzar_hydra <target> 22 100
```

### Resultado esperado vs. obtenido

| Fase | Esperado | Obtenido |
|------|----------|----------|
| **Snort** | Alertas SSH repetidas | 5+ alertas en `/var/log/snort/alert` |
| **Filebeat** | Ingesta automática | Eventos enviados a Logstash |
| **Kibana** | Visualización en Discover | Eventos indexados, filtros por `src_ip` |

---

## 5. Problemas identificados y soluciones

| Incidencia | Causa | Solución |
|------------|-------|----------|
| Regla SSH no detecta intentos | `flow:established` estricto | Cambiar a `not_established` + threshold |
| Filebeat no ingesta alertas | Salida Snort incompatible | `-A fast` + directorio fijo `/var/log/snort` |
| Conectividad Docker | Resolución DNS | Nombre de contenedor `elk` en `filebeat.yml` |