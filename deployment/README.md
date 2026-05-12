# Deployment

Infraestructura como código del sistema. Toda la pila corre sobre **Docker Compose**.

## Contenido

```
deployment/
└── docker/
    ├── .env.example                          # Plantilla de variables de entorno
    ├── nodered/
    │   └── docker-compose.yml                # Node-RED
    └── influxdb-grafana/
        └── docker-compose.yml                # InfluxDB + Grafana + Cloudflared
```

## Servicios

| Contenedor | Imagen | Puerto host | Propósito |
|---|---|---|---|
| `nodered` | `nodered/node-red:latest` | 1880 | Procesamiento MQTT + lógica de alertas |
| `influxdb` | `influxdb:2.7` | 8086 | Almacenamiento de series de tiempo |
| `grafana` | `grafana/grafana:latest` | 3000 | Visualización |
| `cloudflared` | `cloudflare/cloudflared:latest` | — | Túnel seguro para exponer Grafana |

## Despliegue paso a paso

```bash
cd deployment/docker

# 1. Variables de entorno
cp .env.example .env
# Editar .env con valores reales (passwords, tokens)

# 2. Node-RED
docker compose -f nodered/docker-compose.yml --env-file .env up -d

# 3. InfluxDB + Grafana + Cloudflared
docker compose -f influxdb-grafana/docker-compose.yml --env-file .env up -d
```

## Operación

Lista completa de comandos en [`docs/despliegue/comandos-docker.txt`](../docs/despliegue/comandos-docker.txt).

Operaciones básicas:

```bash
docker compose ps                         # Estado
docker compose logs -f <servicio>         # Logs
docker compose restart                    # Reiniciar
docker compose down                       # Detener
```

## Variables de entorno

| Variable | Servicio | Descripción |
|---|---|---|
| `INFLUXDB_USERNAME` | InfluxDB | Usuario administrador inicial |
| `INFLUXDB_PASSWORD` | InfluxDB | Password administrador (≥ 8 caracteres) |
| `INFLUXDB_ORG` | InfluxDB | Organización (por defecto `wsn-heladas`) |
| `INFLUXDB_BUCKET` | InfluxDB | Bucket inicial (por defecto `sensores`) |
| `INFLUXDB_ADMIN_TOKEN` | InfluxDB | Token administrativo inicial |
| `CLOUDFLARED_TOKEN` | Cloudflared | Token del túnel de Cloudflare Zero Trust |
| `TZ` | Todos | Zona horaria (`America/Bogota`) |

## Consideraciones de seguridad

- El archivo `.env` real **no debe versionarse** (está en `.gitignore`).
- Cambiar **siempre** las credenciales de ejemplo antes de exponer servicios.
- Grafana está configurado con **anonymous viewer** y *public dashboards* habilitados;
  desactivarlo si el caso de uso lo requiere.
- Cloudflared crea túnel saliente: no se abren puertos en el firewall.

## Persistencia

Los volúmenes Docker `influxdb-data`, `grafana-data` y la carpeta `nodered/data`
persisten datos entre reinicios. **Hacer respaldos periódicos** (no incluidos en
este repositorio).
