# Descripción de módulos

Cada módulo del repositorio cumple un rol bien delimitado en la arquitectura del sistema.

## `firmware/`

**Capa:** Adquisición + transmisión.
**Lenguaje:** C/C++ (Arduino framework para ESP32).
**Responsable de:** medir variables ambientales y transmitir por LoRaWAN.

| Submódulo | Función |
|---|---|
| `heltec-v3/temp-hum-anemometro-ttn.txt` | Firmware de producción. Lee DHT22, anemómetro de pulsos y batería; transmite vía OTAA cada 15 min. |
| `heltec-v3/medidor-am2305b.txt` | Banco de pruebas del sensor AM2305B (sin LoRaWAN). |
| `heltec-v3/medidor-potencia-led.txt` | Prueba de potencia de transmisión y feedback por LED. |
| `ttn-payload-formatter/heladas-uplink-payload-formatter.txt` | Decoder JavaScript instalado en TTN. Convierte el binario en JSON. |

## `backend/`

**Capa:** Procesamiento + lógica de negocio.
**Tecnología:** Node-RED (Node.js) + MQTT.
**Responsable de:** ingesta MQTT, transformación, almacenamiento y alertas.

| Submódulo | Función |
|---|---|
| `node-red/flows/flows.json` | Definición completa del flujo Node-RED, exportable e importable. |
| `node-red/functions/normalizar-datos.txt` | Aplana el payload anidado de TTN al esquema interno. |
| `node-red/functions/uplink-to-influxdb.txt` | Construye el payload con tipo correcto para InfluxDB; usa `device_id` como *measurement*. |
| `node-red/functions/evaluar-condicion-helada.txt` | Regla de tres condiciones (T, H, V) para disparar alerta de helada. |

## `frontend/`

**Capa:** Presentación al usuario final.
**Tecnología:** HTML + JS + Leaflet.js (sin build).
**Responsable de:** mostrar mapa de nodos y embeber dashboards de Grafana.

| Archivo | Función |
|---|---|
| `index.html` | SPA estática. Mapa Leaflet con marcadores por nodo, popups con telemetría y embebido de Grafana. |

## `deployment/`

**Capa:** Infraestructura.
**Tecnología:** Docker Compose.
**Responsable de:** orquestación de Node-RED, InfluxDB, Grafana y Cloudflared.

| Submódulo | Función |
|---|---|
| `docker/nodered/docker-compose.yml` | Node-RED (puerto 1880). |
| `docker/influxdb-grafana/docker-compose.yml` | InfluxDB 2.7 + Grafana + túnel Cloudflared. |
| `docker/.env.example` | Plantilla de credenciales. |

## `docs/`

**Capa:** Documentación.
Markdown + diagramas. No participa en runtime. Aporta trazabilidad académica y operativa.

## `scripts/`

Reservado para scripts utilitarios futuros (backups de InfluxDB, exportación de
datos, automatización de despliegue, etc.). Actualmente vacío.
