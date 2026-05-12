# Software LoRaWAN Heladas

**Sistema IoT/LoRaWAN para monitoreo y generación de alertas tempranas de heladas en cultivos de papa**

![Status](https://img.shields.io/badge/status-active-success)
![Platform](https://img.shields.io/badge/platform-Heltec_WiFi_LoRa_32_V3-blue)
![Network](https://img.shields.io/badge/network-LoRaWAN%20%7C%20TTN-orange)
![Stack](https://img.shields.io/badge/stack-Node--RED%20%7C%20InfluxDB%20%7C%20Grafana-purple)
![License](https://img.shields.io/badge/license-Apache_2.0-lightgrey)

---

## Tabla de contenido

- [Descripción general](#descripción-general)
- [Autores y entidad](#autores-y-entidad)
- [Arquitectura del sistema](#arquitectura-del-sistema)
- [Tecnologías utilizadas](#tecnologías-utilizadas)
- [Estructura del repositorio](#estructura-del-repositorio)
- [Despliegue rápido](#despliegue-rápido)
- [Variables de entorno](#variables-de-entorno)
- [Branches del repositorio](#branches-del-repositorio)
- [Seguridad y registro DNDA](#seguridad-y-registro-dnda)
- [Licencia](#licencia)

---

## Descripción general

Este software implementa una **red inalámbrica de sensores (WSN)** basada en
tecnología **LoRaWAN** para la adquisición, transmisión, procesamiento, almacenamiento,
visualización y notificación de variables ambientales relacionadas con eventos de
**helada** en cultivos de papa de zonas alto-andinas.

El sistema captura **temperatura**, **humedad relativa** y **velocidad del viento**
en cada nodo sensor; transmite los datos por LoRaWAN hacia *The Things Network* (TTN);
los procesa mediante flujos de Node-RED; los almacena en InfluxDB; los visualiza en
dashboards de Grafana y genera **alertas automáticas** (WhatsApp vía Twilio) cuando
las condiciones meteorológicas se aproximan a un umbral de helada.

Las redes desplegadas se encuentran en **La Calera (Cundinamarca)** y **Turmequé (Boyacá)**.

---

## Autores y entidad

| Rol | Nombre |
|---|---|
| Autor | David Nicolás Murillo Nova |
| Autor | Philip Mateo Millán Patiño |

Proyecto desarrollado en el marco del **grupo de investigación LIDER**, adscrito a la
**Facultad de Ingeniería de la Universidad Distrital Francisco José de Caldas (UDFJC)**.

---

## Arquitectura del sistema

```
┌────────────────────┐    LoRaWAN    ┌─────────────┐    MQTT     ┌────────────┐
│  Nodos Heltec V3   │ ────────────► │     TTN     │ ──────────► │  Node-RED  │
│  (DHT22 / AM2305B  │   868/915MHz  │ (gateway +  │  v3 broker  │  (flujos   │
│   anemómetro)      │               │  app)       │             │   JSON)    │
└────────────────────┘               └─────────────┘             └─────┬──────┘
                                                                       │
                                              ┌────────────────────────┼────────────────────┐
                                              ▼                        ▼                    ▼
                                       ┌─────────────┐         ┌─────────────┐       ┌─────────────┐
                                       │  InfluxDB   │         │  Algoritmo  │       │   Grafana   │
                                       │  series de  │         │  evaluación │       │  dashboards │
                                       │   tiempo    │         │  de helada  │       │  + público  │
                                       └─────────────┘         └──────┬──────┘       └──────┬──────┘
                                                                      │                     │
                                                                      ▼                     ▼
                                                              ┌─────────────┐       ┌──────────────────┐
                                                              │   Twilio    │       │  Frontend web    │
                                                              │  WhatsApp   │       │  (Leaflet + iframe│
                                                              │   alertas   │       │   Grafana)       │
                                                              └─────────────┘       └──────────────────┘

                                                                          ┌──────────────────┐
                                              Acceso remoto seguro vía ──►│  Cloudflared     │
                                                                          │  tunnel          │
                                                                          └──────────────────┘
```

Más detalle en [`docs/arquitectura/`](docs/arquitectura/).

---

## Tecnologías utilizadas

| Capa | Tecnología | Uso |
|---|---|---|
| Hardware | Heltec WiFi LoRa 32 V3 (ESP32-S3 + SX1262) | Nodos sensores |
| Sensores | DHT22 / AM2305B | Temperatura y humedad |
| Sensores | Anemómetro de pulsos | Velocidad del viento |
| Radio | LoRaWAN (Class A, OTAA) | Transmisión inalámbrica |
| Red | The Things Network (TTN v3) | Network Server |
| Procesamiento | Node-RED + MQTT | Flujos de datos y lógica |
| Almacenamiento | InfluxDB 2.7 | Base de datos de series de tiempo |
| Visualización | Grafana | Dashboards en tiempo real |
| Alertas | Twilio WhatsApp API | Notificaciones a usuarios |
| Frontend | HTML + Leaflet.js | Mapa de nodos y portal |
| Acceso remoto | Cloudflared Tunnel | Túnel seguro sin puertos abiertos |
| Despliegue | Docker + Docker Compose | Orquestación local |

Detalle completo en [`docs/arquitectura/tecnologias.md`](docs/arquitectura/tecnologias.md).

---

## Estructura del repositorio

```
Software_LoRaWAN_Heladas/
├── firmware/                  # Código de los nodos sensores y formateador TTN
│   ├── heltec-v3/
│   └── ttn-payload-formatter/
├── backend/                   # Lógica de procesamiento de datos
│   └── node-red/
│       ├── flows/
│       └── functions/
├── frontend/                  # Interfaz web pública (GitHub Pages)
├── deployment/                # Infraestructura como código
│   └── docker/
│       ├── nodered/
│       ├── influxdb-grafana/
│       └── .env.example
├── docs/                      # Documentación técnica
│   ├── arquitectura/
│   ├── despliegue/
│   ├── diagramas/
│   ├── pruebas/
│   ├── tesis/
│   └── imagenes/
├── scripts/                   # Scripts utilitarios (reservado)
├── .gitignore
├── LICENSE
└── README.md
```

Cada módulo tiene su propio `README.md` con instrucciones específicas.

---

## Despliegue rápido

> Requiere **Docker** y **Docker Compose** instalados.

```bash
# 1. Configurar variables de entorno
cd deployment/docker
cp .env.example .env
# Editar .env con credenciales reales

# 2. Levantar Node-RED
docker compose -f nodered/docker-compose.yml --env-file .env up -d

# 3. Levantar InfluxDB + Grafana + Cloudflared
docker compose -f influxdb-grafana/docker-compose.yml --env-file .env up -d
```

Servicios disponibles:

| Servicio | URL local |
|---|---|
| Node-RED | http://localhost:1880 |
| InfluxDB | http://localhost:8086 |
| Grafana | http://localhost:3000 |

Lista completa de comandos en [`docs/despliegue/comandos-docker.txt`](docs/despliegue/comandos-docker.txt).

---

## Variables de entorno

Ver [`deployment/docker/.env.example`](deployment/docker/.env.example).
**Nunca** versionar el archivo `.env` real (ya está en `.gitignore`).

---

## Branches del repositorio

| Branch | Propósito |
|---|---|
| `main` | Rama estable, contiene la última versión integrada |
| `firmware` | Trabajo activo sobre el código de los nodos Heltec |
| `backend` | Trabajo activo sobre Node-RED, flujos y funciones |
| `frontend` | Trabajo activo sobre la interfaz web |
| `deployment` | Trabajo activo sobre Docker, configuración e infraestructura |
| `docs` | Trabajo activo sobre documentación y diagramas |

---

## Seguridad y registro DNDA

Para efectos de registro ante la **Dirección Nacional de Derecho de Autor (DNDA)**,
este repositorio se ha publicado eliminando credenciales, tokens y configuraciones
sensibles. Los valores que aparecen en los archivos de configuración son **ejemplos**
y deben ser reemplazados antes de poner el sistema en producción.

---

## Licencia

Este software se distribuye bajo la licencia **Apache License 2.0**.
Consulta el archivo [LICENSE](LICENSE) para más detalles.
