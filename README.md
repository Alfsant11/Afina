# Afina

Motor de recomendación por similitud multidimensional para descubrir entidades afines, generar agrupaciones y construir listas personalizadas a partir de atributos numéricos.

> **Estado:** especificación completa; implementación pendiente.

## Descripción

Afina está diseñado para analizar colecciones de canciones, películas, ítems de juego u otras entidades representadas mediante múltiples características numéricas. El sistema normaliza y pondera esas características para comparar entidades de forma consistente.

El producto ofrecerá tres tipos de resultados independientes:

- **Recomendaciones por entidad:** vecinos ordenados respecto de una referencia.
- **Clusters de afinidad:** partición global de todas las entidades aptas, sin significado de ranking.
- **Listas curadas:** selección ordenada respecto de un perfil creado con las preferencias del usuario y filtros opcionales.

## Funcionalidades previstas

- Importación incremental de archivos JSON y CSV.
- Validación estructurada de esquemas, identificadores y valores numéricos.
- Normalización min-max y estandarización Z poblacional.
- Pesos configurables por característica.
- Distancia euclidiana y similitud coseno.
- Recomendaciones top-K deterministas.
- Clustering Lloyd para distancia euclidiana.
- K-means esférico para similitud coseno.
- Reparación determinista de clusters vacíos.
- Representantes, medias y coeficiente de silueta por agrupación.
- Explicaciones mediante contribuciones por característica.
- Filtros de elegibilidad para listas personalizadas.
- Exportación determinista a JSON y CSV.
- Procesamiento local sin enviar datasets a servicios externos.

## Arquitectura prevista

```text
Interfaz React + TypeScript
          │
          ▼
Gateway de análisis
          │
          ▼
Web Workers ── Rust/WebAssembly
                    │
                    ├── Importación y validación
                    ├── Normalización y métricas
                    ├── Recomendaciones y listas
                    ├── Clustering y explicabilidad
                    └── Serialización
          │
          ▼
IndexedDB local
```

Tecnologías principales:

- React, TypeScript estricto y Vite para la aplicación web.
- Rust para el núcleo matemático.
- WebAssembly para ejecutar el núcleo en el navegador.
- Web Workers para mantener los cálculos fuera del hilo principal.
- IndexedDB para configuración y últimos resultados válidos.
- `proptest` y Playwright para validación automatizada.

## Principios del proyecto

### Local-first y privacidad

Los datasets y resultados deben permanecer en el dispositivo del usuario. La aplicación no incluirá telemetría, sincronización remota, clientes HTTP ni dependencias de servicios externos durante la ejecución.

### Determinismo

La misma colección, configuración, semilla y versión deben producir los mismos resultados. Los empates se resuelven mediante identificadores ordenados por Unicode.

### Exactitud matemática

No se introducirán aproximaciones silenciosas. Si el cálculo exacto de representantes o silueta no alcanza los objetivos de rendimiento, se revisarán explícitamente los requisitos antes de cambiar el comportamiento.

### Estado transaccional

Un resultado nuevo solo sustituirá al anterior cuando el análisis termine y supere todas las verificaciones. Los errores, cancelaciones o fallos de workers no deben publicar resultados parciales.

## Estructura actual

```text
.kiro/
├── specs/
│   └── motor-recomendacion-similitud/
│       ├── requirements.md
│       ├── design.md
│       └── tasks.md
└── steering/
    └── project.md
```

## Documentación

- [Requisitos](.kiro/specs/motor-recomendacion-similitud/requirements.md)
- [Diseño técnico](.kiro/specs/motor-recomendacion-similitud/design.md)
- [Plan de implementación](.kiro/specs/motor-recomendacion-similitud/tasks.md)
- [Reglas del proyecto](.kiro/steering/project.md)

## Cómo continuar

El código de la aplicación todavía no ha sido generado. La implementación debe seguir el orden y las dependencias del [plan de tareas](.kiro/specs/motor-recomendacion-similitud/tasks.md), comenzando por la preparación de los workspaces web y Rust.

Antes de considerar completa cada tarea se deben ejecutar las pruebas dirigidas, formato, lint, comprobación de tipos y builds de los paquetes afectados.

## Objetivos de capacidad

El diseño contempla colecciones de hasta 100.000 entidades y 100 características numéricas dentro del entorno de referencia definido en los requisitos. El cumplimiento de los objetivos de tiempo y memoria deberá demostrarse mediante benchmarks locales reproducibles.