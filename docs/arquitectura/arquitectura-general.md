# Arquitectura general del sistema

## Resumen

Sistema distribuido de **adquisición, transmisión, procesamiento y visualización**
de variables ambientales para la detección temprana de heladas agrícolas en cultivos
de papa de zonas alto-andinas (Cundinamarca y Boyacá, Colombia).

El sistema sigue una arquitectura de **5 capas**:

1. **Capa de adquisición** – sensores físicos y microcontrolador.
2. **Capa de transmisión** – LoRaWAN sobre TTN.
3. **Capa de procesamiento** – Node-RED + lógica de negocio.
4. **Capa de almacenamiento y visualización** – InfluxDB + Grafana.
5. **Capa de presentación y notificación** – frontend web y alertas WhatsApp.

---

## Diagrama de bloques

```
┌──────────────────────────────────────────────────────────────────────────┐
│ CAPA 1 · ADQUISICIÓN                                                     │
│                                                                          │
│   ┌────────────┐  ┌────────────┐  ┌────────────┐                         │
│   │  DHT22 /   │  │ Anemómetro │  │  ADC       │                         │
│   │  AM2305B   │  │  pulsos    │  │  batería   │                         │
│   └─────┬──────┘  └─────┬──────┘  └─────┬──────┘                         │
│         │               │               │                                │
│         ▼               ▼               ▼                                │
│   ┌──────────────────────────────────────────────┐                       │
│   │   Heltec WiFi LoRa 32 V3 (ESP32-S3 + SX1262) │                       │
│   │   Deep sleep 15 min · contador RTC           │                       │
│   └─────────────────────┬────────────────────────┘                       │
└─────────────────────────┼────────────────────────────────────────────────┘
                          │
                          │ LoRaWAN (Class A, OTAA)
                          ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ CAPA 2 · TRANSMISIÓN                                                     │
│   ┌─────────────┐    ┌──────────────────┐                                │
│   │  Gateway    │ ──►│   TTN v3         │  ──► MQTT broker               │
│   │  LoRa       │    │  Network Server  │                                │
│   └─────────────┘    └──────────────────┘                                │
└─────────────────────────┼────────────────────────────────────────────────┘
                          │ MQTT (v3/<app>@<tenant>/devices/+/up)
                          ▼
┌──────────────────────────────────────────────────────────────────────────┐
│ CAPA 3 · PROCESAMIENTO (Node-RED)                                        │
│                                                                          │
│   [mqtt in] → [normalizar-datos] → [uplink-to-influxdb] → [InfluxDB out] │
│                       │                                                  │
│                       └→ [evaluar-condicion-helada] → [Twilio config]    │
│                                                       → [http request]   │
└─────────────────────────┼────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
┌─────────────┐   ┌─────────────┐   ┌─────────────┐
│  InfluxDB   │   │   Twilio    │   │  Grafana    │
│  2.7        │   │  WhatsApp   │   │  dashboards │
└─────────────┘   └─────────────┘   └──────┬──────┘
                                           │ iframes públicos
                                           ▼
                              ┌──────────────────────────┐
                              │  Frontend (GitHub Pages) │
                              │  Mapa Leaflet + iframe   │
                              └──────────────────────────┘
```

---

## Flujo de datos detallado

### 1. Adquisición y empaquetado

Cada nodo despierta del *deep sleep* cada 15 minutos:

1. Lee `temperatura` y `humedad` del DHT22/AM2305B.
2. Si tiene anemómetro, cuenta pulsos durante 3 s.
3. Lee la tensión de batería.
4. Incrementa el contador de paquete (persistido en RTC).
5. Empaqueta el binario (8 o 10 bytes según haya anemómetro).
6. Transmite por LoRaWAN (Class A, OTAA).
7. Vuelve a *deep sleep*.

### 2. Transmisión LoRaWAN → TTN

El gateway captura la trama y la entrega al *Network Server* de TTN.
TTN aplica el **Payload Formatter** ([decoder JS](../../firmware/ttn-payload-formatter/heladas-uplink-payload-formatter.txt))
y publica el JSON en su broker MQTT v3.

### 3. Procesamiento Node-RED

El flujo se compone de:

- **mqtt in** → suscrito al topic `v3/<app>@<tenant>/devices/+/up`.
- **normalizar-datos** → aplana el payload TTN a un esquema interno con `Device_id`,
  `Temperatura`, `Humedad`, `Bateria`, `Viento`, `Paquete`, `timestamp`.
- **uplink-to-influxdb** → construye el payload de medición usando el `device_id` como
  `measurement` (un nodo por *measurement*).
- **evaluar-condicion-helada** → si `T ≤ 2°C ∧ H ≤ 50% ∧ V ≤ 5 m/s`, genera el mensaje
  de alerta.
- **Twilio config + http request** → publica la alerta vía la API de Twilio WhatsApp.

### 4. Almacenamiento y visualización

- **InfluxDB 2.7**: organización `wsn-heladas`, bucket `sensores`, retención infinita.
- **Grafana**: dashboards por nodo y por red (La Calera, Turméque). Cada dashboard se
  publica como *Public Dashboard* y se embebe vía iframe en el frontend.

### 5. Presentación

- **Frontend web** (`frontend/index.html`): mapa Leaflet con marcadores por nodo,
  popups con últimas lecturas y enlace a cada dashboard.
- **Acceso remoto**: Grafana se expone públicamente a través de un túnel Cloudflared
  (sin abrir puertos en el firewall del servidor).

---

## Consideraciones de diseño

- **Bajo consumo**: deep sleep entre ventanas de medición; el firmware está pensado
  para operación con batería y panel solar.
- **Resiliencia**: contador de paquetes persistido en RTC; reconexión OTAA automática.
- **Escalabilidad horizontal**: agregar nuevos nodos solo requiere registrarlos en TTN.
  El flujo de Node-RED usa el `device_id` dinámicamente como *measurement*.
- **Separación de zonas**: cada red (La Calera / Turméque) se identifica por convención
  en los `device_id` y se visualiza en dashboards independientes.
