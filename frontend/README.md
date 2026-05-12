# Frontend

Interfaz web pública del sistema (mapa de nodos + dashboards integrados de Grafana).

## Contenido

```
frontend/
└── index.html        # SPA estática, sin build, ~1 archivo
```

## Características

- **Mapa interactivo** con Leaflet.js de los nodos LoRaWAN desplegados.
- **Dos redes** preconfiguradas:
  - **Red La Calera** (Cundinamarca) – 4 nodos.
  - **Red Turméque** (Boyacá) – 2 nodos.
- **Popup por nodo** con telemetría reciente (temperatura, humedad, viento, batería, SNR, RSSI).
- **Iframe a dashboards públicos de Grafana** por nodo y por red.
- **Diseño dark con tema sobrio**, fuentes Space Mono / DM Sans.

## Configuración de URLs de Grafana

Las URLs públicas de Grafana se definen en el bloque `URLS` dentro de `index.html`:

```js
const GRAFANA_BASE = 'https://grafana.lorawannetworksud.com';

const URLS = {
  'CALERA1': GRAFANA_BASE + '/public-dashboards/<UID>',
  // ...
};
```

Para obtener cada UID: en Grafana → abrir dashboard → **Share → Public dashboard → Copy link**.

## Despliegue

Es una página estática. Opciones:

1. **GitHub Pages** (recomendado): habilitar Pages sobre la branch `main` y la carpeta
   `/frontend` o servir desde una branch `gh-pages` específica.
2. **Cualquier servidor HTTP estático** (`python -m http.server`, `nginx`, etc.).

## Dependencias externas (CDN)

- Leaflet 1.9.4
- Google Fonts (Space Mono, DM Sans)
- OpenStreetMap tiles

No hay build paso. Editar el HTML directamente.
