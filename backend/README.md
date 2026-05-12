# Backend

Capa de procesamiento, almacenamiento y lógica de negocio del sistema.

## Contenido

```
backend/
└── node-red/
    ├── flows/
    │   └── flows.json                       # Flujo principal exportado de Node-RED
    └── functions/
        ├── normalizar-datos.txt             # Aplana payload TTN a esquema interno
        ├── uplink-to-influxdb.txt           # Construye payload para nodo InfluxDB
        └── evaluar-condicion-helada.txt     # Evalúa umbral y produce mensaje de alerta
```

## Flujo de datos

```
MQTT TTN (v3/<app>@<tenant>/devices/+/up)
        │
        ▼
  [normalizar-datos]  ──► Device_id, Temperatura, Humedad, Bateria, Viento, Paquete, timestamp
        │
        ├──► [uplink-to-influxdb] ──► InfluxDB (measurement = device_id)
        │
        └──► [evaluar-condicion-helada] ──► si T≤2°C ∧ H≤50% ∧ V≤5 m/s
                                                  │
                                                  └──► Twilio WhatsApp (alerta)
```

## Topic MQTT de TTN

```
v3/<application-id>@<tenant-id>/devices/+/up
```

## Función `evaluar-condicion-helada`

Condición de helada (regla simple validada con el grupo LIDER):

```
Temperatura ≤ 2 °C   ∧   Humedad ≤ 50 %   ∧   Viento ≤ 5 m/s
```

Cumplir las tres condiciones simultáneamente dispara la alerta.

## Despliegue

Node-RED corre como contenedor Docker. Ver
[`deployment/docker/nodered/`](../deployment/docker/nodered/).

1. Levantar el contenedor:
   ```bash
   cd deployment/docker
   docker compose -f nodered/docker-compose.yml --env-file .env up -d
   ```
2. Acceder a Node-RED en `http://localhost:1880`.
3. Importar el flujo desde **Menu → Import → select file →
   `backend/node-red/flows/flows.json`**.
4. Configurar las credenciales del broker MQTT de TTN y del nodo de salida InfluxDB.

> ⚠ Las credenciales (Twilio SID/Token, InfluxDB token, MQTT API key de TTN) **no
> están versionadas**; se configuran localmente en cada nodo de Node-RED.
