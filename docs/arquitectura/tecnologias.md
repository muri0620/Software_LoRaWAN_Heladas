# Tecnologías utilizadas

Stack completo del sistema, con versión y propósito.

## Hardware

| Componente | Modelo | Notas |
|---|---|---|
| Microcontrolador | Heltec WiFi LoRa 32 V3 | ESP32-S3 + radio Semtech SX1262 + OLED 128x64 |
| Sensor temperatura/humedad | DHT22 / AM2305B | Precisión ±0.5 °C, ±2 % HR |
| Anemómetro | De pulsos (reed switch) | Lectura por interrupción ISR, ventana 3 s |
| Batería | LiPo 3.7 V | Carga vía USB-C + Modulo de carga Heltec LoRa 32 V3 |

## Comunicación

| Tecnología | Versión / Banda | Uso |
|---|---|---|
| LoRaWAN | Class A, OTAA | Capa física + acceso al medio |
| The Things Network | v3 | Network Server público |
| MQTT | 3.1.1 (TTN v3) | Salida de TTN hacia Node-RED |

## Software

| Componente | Versión | Imagen / Repo | Uso |
|---|---|---|---|
| Arduino framework (ESP32) | core 3.x | `espressif/arduino-esp32` | Firmware |
| `heltec_unofficial` | última | repo de Heltec | BSP del Heltec V3 |
| `LoRaWAN_ESP32` | última | repo de Heltec | Stack LoRaWAN |
| `DHT sensor library` | 1.4+ | `adafruit/DHT-sensor-library` | Lectura DHT22 |
| Node-RED | latest | `nodered/node-red:latest` | Flujos de datos |
| InfluxDB | 2.7 | `influxdb:2.7` | Series de tiempo |
| Grafana | latest | `grafana/grafana:latest` | Visualización |
| Cloudflared | latest | `cloudflare/cloudflared:latest` | Túnel seguro |
| Twilio WhatsApp API | sandbox/2010-04-01 | – | Notificaciones |
| Leaflet | 1.9.4 | CDN unpkg | Mapa frontend |
| Docker | 24.x+ | – | Orquestación |
| Docker Compose | v2 | – | Definición declarativa de servicios |

## Plan de frecuencias

Colombia opera en la banda **AU915 / US915** según el plan acordado con TTN. Verificar
en TTN Console → Application → End device → *Frequency plan*.

## Topologías

- **Red La Calera** (Cundinamarca): 4 nodos, gateway local.
- **Red Turméque** (Boyacá): 2 nodos, gateway local.

Ambas redes comparten la misma aplicación TTN.
