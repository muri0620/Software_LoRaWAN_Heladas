# Firmware

Código embebido para los nodos sensores y el formateador de payload de TTN.

## Contenido

```
firmware/
├── heltec-v3/                                # Sketches para Heltec WiFi LoRa 32 V3
│   ├── temp-hum-anemometro-ttn.txt           # Firmware de producción (3 variables + LoRaWAN)
│   ├── medidor-am2305b.txt                   # Banco de prueba del sensor DHT/AM2305B
│   └── medidor-potencia-led.txt              # Prueba de potencia de transmisión + LED
└── ttn-payload-formatter/
    └── heladas-uplink-payload-formatter.txt  # Decoder JavaScript para TTN
```

## Firmware principal

[`heltec-v3/temp-hum-anemometro-ttn.txt`](heltec-v3/temp-hum-anemometro-ttn.txt)

- **MCU:** ESP32-S3 (Heltec WiFi LoRa 32 V3)
- **Radio:** SX1262 (868/915 MHz, dependiendo del plan regional)
- **Sensores:**
  - DHT22 / AM2305B (pin 46) – temperatura y humedad
  - Anemómetro de pulsos (pin 47, interrupción) – velocidad del viento
  - ADC de batería (pin 1)
- **Activación LoRaWAN:** OTAA, Class A
- **Periodo de envío:** 15 minutos (`MINIMUM_DELAY`)
- **Persistencia:** contador de paquetes en RTC (sobrevive *deep sleep*)
- **Anti-rebote ISR:** 10 ms

### Payload (binario, 8 o 10 bytes)

| Offset | Bytes | Campo | Escala |
|---|---|---|---|
| 0 | 2 | `packet` (contador) | – |
| 2 | 2 | `temperature` (int16, signed) | / 100 |
| 4 | 2 | `humidity` (uint16) | / 100 |
| 6 | 2 | `battery` (uint16) | / 100 |
| 8 | 2 | `wind_speed` (uint16, opcional) | / 100 |

Los nodos sin anemómetro envían 8 bytes; los que lo tienen, 10.

## TTN Payload Formatter

Decoder JavaScript que se pega en *The Things Network → Application → Payload formatters → Uplink*.
Convierte el binario anterior en JSON consumible por Node-RED.

## Dependencias (Arduino IDE / PlatformIO)

- `heltec_unofficial` (board support)
- `LoRaWAN_ESP32`
- `DHT sensor library` (Adafruit)
- `Adafruit Unified Sensor`

## Compilación

1. Instalar el core de Heltec para ESP32 (Boards Manager).
2. Seleccionar **Heltec WiFi LoRa 32 (V3)**.
3. Configurar las claves OTAA (`DevEUI`, `AppEUI`, `AppKey`) según el dispositivo
   registrado en TTN.
4. Cargar el sketch correspondiente.

> ⚠ Las claves OTAA **no se incluyen en este repositorio**; deben configurarse
> localmente en cada nodo.
