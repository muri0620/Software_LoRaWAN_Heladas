# Documentación

Documentación técnica, académica y operativa del proyecto.

## Estructura

```
docs/
├── arquitectura/         # Diseño general, módulos, tecnologías
├── despliegue/           # Guías operativas y comandos
├── diagramas/            # Diagramas (flujo de datos, red, hardware)
├── pruebas/              # Protocolos y resultados de pruebas de campo
├── tesis/                # Documento de tesis y material académico
└── imagenes/             # Fotografías de los nodos y despliegue
```

## Documentos disponibles

- [Arquitectura general](arquitectura/arquitectura-general.md) – flujo de datos completo del sistema.
- [Descripción de módulos](arquitectura/descripcion-modulos.md) – qué hace cada subsistema.
- [Tecnologías utilizadas](arquitectura/tecnologias.md) – stack completo con versiones.
- [Comandos Docker](despliegue/comandos-docker.txt) – operación cotidiana del sistema.

## Convenciones

- Documentos en **español**.
- Markdown (`.md`) para texto, **kebab-case** para nombres de archivo.
- Imágenes en `imagenes/`, referenciadas con rutas relativas.
- Diagramas exportables a SVG/PNG; mantener fuentes editables (`.drawio`, `.excalidraw`) si las hay.
