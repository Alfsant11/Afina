# Implementation Plan: Motor de recomendación y similitud

## Overview

Este plan construye incrementalmente una aplicación web local-first con React, TypeScript y Vite, respaldada por un núcleo Rust compilable como biblioteca nativa y WebAssembly. El orden prioriza contratos, dominio matemático y benchmarks de riesgo antes de invertir en la interfaz completa. Cada paso produce código integrado; los resultados analíticos se publican de forma transaccional y todo el procesamiento permanece en el dispositivo.

## Tasks

- [ ] 1. Preparar el workspace y las cadenas de herramientas
  - [ ] 1.1 Crear el proyecto web React + TypeScript + Vite y la estructura de paquetes
    - Crear `apps/web` con configuración estricta de TypeScript, scripts no interactivos de build, lint y test, y directorios para UI, gateway, workers y persistencia.
    - Fijar versiones exactas de dependencias y evitar clientes de red o telemetría.
    - _Requisitos: 8.4, 8.8_
  - [ ] 1.2 Crear el workspace Rust y los crates del núcleo
    - Crear crates separados para dominio puro, importación/exportación, casos de uso, bindings Wasm y benchmarks nativos.
    - Configurar `f64`, features nativa/Wasm y perfiles reproducibles sin dependencias de red en runtime.
    - _Requisitos: 8.4, 8.7, 8.8_
  - [ ] 1.3 Integrar la compilación Rust/Wasm con Vite
    - Añadir scripts reproducibles para compilar el núcleo nativo, generar bindings `wasm-bindgen` y empaquetar Wasm y workers como assets locales.
    - Verificar mediante un smoke test que TypeScript puede cargar una función mínima del módulo Wasm.
    - _Requisitos: 8.4, 8.8_
  - [ ] 1.4 Configurar los arneses de pruebas automatizadas
    - Configurar pruebas Rust, `proptest`, pruebas TypeScript/React, integración Wasm y Playwright offline; cada comando debe terminar sin modo watch.
    - Preparar utilidades compartidas de fixtures deterministas y semillas reproducibles.
    - _Requisitos: 4.17, 8.7, 8.8_

- [ ] 2. Definir contratos canónicos y errores entre capas
  - [ ] 2.1 Implementar los modelos y contratos TypeScript
    - Crear tipos para esquema, dataset, validación, transformación, solicitudes, resultados, páginas, exportación y unión discriminada del protocolo worker.
    - Modelar explícitamente presencia, nulo, vacío y ausencia en metadatos.
    - _Requisitos: 1.9, 6.8, 7.7, 7.8_
  - [ ] 2.2 Implementar los modelos de dominio Rust
    - Crear modelos equivalentes con matrices row-major, IDs canonizados, column store, configuración efectiva, resultados y estados de ausencia.
    - Aplicar invariantes de finitud y tipos mediante constructores validados.
    - _Requisitos: 1.3, 1.6, 2.9, 6.8, 7.7, 7.8_
  - [ ] 2.3 Implementar el contrato binario TypeScript–Wasm
    - Definir descriptores de buffers, arreglos tipados y tablas de offsets para evitar cruzar objetos por entidad durante cálculos masivos.
    - Añadir validación de versión de contrato y tests de codificación/decodificación.
    - _Requisitos: 8.1, 8.2, 8.3, 8.7_
  - [ ] 2.4 Implementar la taxonomía estable de errores y versión del sistema
    - Crear códigos, categorías, localización, datos seguros y conversión Rust/TypeScript sin incluir filas ni vectores completos.
    - Incorporar `systemVersion`, `requestId` y errores recuperables en todos los límites públicos.
    - _Requisitos: 1.9, 2.8, 3.7, 3.11, 4.5, 8.6, 8.7_

- [ ] 3. Implementar importación incremental y validación
  - [ ] 3.1 Implementar la validación del esquema declarado
    - Validar nombres, orden, roles, tipos, obligatoriedad, representación de nulo/vacío/ausente, ruta JSON y encabezados CSV.
    - Clasificar como global cualquier condición que impida interpretar registros de forma fiable.
    - _Requisitos: 1.1, 1.2, 1.8_
  - [ ] 3.2 Implementar el importador JSON incremental
    - Consumir bloques UTF-8, exigir un documento RFC 8259, localizar la colección declarada y producir borradores de entidad sin cargar copias innecesarias.
    - Mantener posición de registro y estados de metadatos para round-trip.
    - _Requisitos: 1.1, 1.6, 1.8, 7.7, 8.1_
  - [ ] 3.3 Implementar el importador CSV incremental
    - Parsear RFC 4180 con comillas, saltos y CRLF; validar encabezados recortados únicos/no vacíos y una característica como mínimo.
    - Emitir una entidad por fila no vacía y preservar las codificaciones declaradas de valores ausentes.
    - _Requisitos: 1.2, 1.6, 1.8, 7.8, 8.1_
  - [ ] 3.4 Implementar la canalización de validación y el informe
    - Canonizar IDs, validar longitud y números finitos, excluir entidades completas con errores y rechazar todas las apariciones de IDs duplicados.
    - Generar informe determinista con ubicación, causa, conteos completos y límite seguro de detalles; producir dataset vacío ante error global o cero entidades válidas.
    - _Requisitos: 1.3–1.10_
  - [ ] 3.5 Escribir la prueba basada en propiedades de correspondencia JSON
    - Crear un archivo de prueba `proptest` dedicado con al menos 100 casos exitosos.
    - **Propiedad 1: Correspondencia de registros JSON válidos**
    - **Valida: Requisito 1.1**
  - [ ] 3.6 Escribir la prueba basada en propiedades de correspondencia CSV
    - Cubrir filas vacías y encabezados recortados válidos e inválidos en un archivo de propiedad independiente.
    - **Propiedad 2: Correspondencia y encabezados CSV**
    - **Valida: Requisito 1.2**
  - [ ] 3.7 Escribir la prueba basada en propiedades de identificadores
    - Generar Unicode, espacios de borde y longitudes 0, 1, 255 y 256.
    - **Propiedad 3: Canonización y límites de identificadores**
    - **Valida: Requisito 1.3**
  - [ ] 3.8 Escribir la prueba basada en propiedades de unicidad
    - Generar colecciones con colisiones posteriores al recorte y comprobar que se rechazan todas las apariciones.
    - **Propiedad 4: Unicidad de identificadores canonizados**
    - **Valida: Requisitos 1.4, 1.5**
  - [ ] 3.9 Escribir la prueba basada en propiedades de aislamiento de entidades
    - Generar números finitos y valores inválidos explícitos, incluidos booleanos, NaN e infinitos.
    - **Propiedad 5: Aislamiento de entidades inválidas**
    - **Valida: Requisitos 1.6, 1.7, 1.10**
  - [ ] 3.10 Escribir la prueba basada en propiedades del informe
    - Recalcular de forma independiente la partición aceptada/rechazada y las ubicaciones de error.
    - **Propiedad 6: Exactitud del informe de validación**
    - **Valida: Requisito 1.9**
  - [ ] 3.11 Escribir pruebas unitarias de fronteras de importación
    - Cubrir JSON/CSV malformado, BOM, cero válidas, error global, truncado de detalles, Unicode y combinación de errores por entidad.
    - _Requisitos: 1.1–1.10, 7.1, 7.2_

- [ ] 4. Implementar normalización, pesos y configuración efectiva
  - [ ] 4.1 Implementar el normalizador numérico
    - Calcular min-max y Z poblacional en dos pasadas con Welford, usando cero para columnas constantes y `N=1`.
    - Conservar matriz original y normalizada; calcular la vista ponderada sin duplicar toda la matriz.
    - _Requisitos: 2.1–2.5, 8.1_
  - [ ] 4.2 Implementar validación y commit atómico de pesos
    - Aplicar peso predeterminado 1, admitir cero y rechazar de forma atómica pesos negativos, no numéricos o no finitos con localización de característica.
    - Mantener sin cambios la última configuración válida ante error.
    - _Requisitos: 2.5–2.8_
  - [ ] 4.3 Implementar parámetros y caché de transformación
    - Asociar método, estadísticos, todos los pesos efectivos, métrica, semilla y versión a una configuración inmutable.
    - Cachear por huella de dataset y método sin confundir configuraciones ni mutar vectores normalizados.
    - _Requisitos: 2.7, 2.9, 6.8, 8.7_
  - [ ] 4.4 Escribir la prueba basada en propiedades min-max
    - **Propiedad 7: Normalización min-max**
    - **Valida: Requisito 2.1**
  - [ ] 4.5 Escribir la prueba basada en propiedades de estandarización
    - Usar generadores constructivos bien condicionados y tolerancia absoluta `1e-6`.
    - **Propiedad 8: Estandarización poblacional**
    - **Valida: Requisitos 2.2, 2.3**
  - [ ] 4.6 Escribir la prueba basada en propiedades de columnas constantes
    - **Propiedad 9: Columnas constantes se transforman en cero**
    - **Valida: Requisito 2.4**
  - [ ] 4.7 Escribir la prueba basada en propiedades de ponderación
    - **Propiedad 10: Ponderación componente a componente**
    - **Valida: Requisitos 2.5, 2.7**
  - [ ] 4.8 Escribir la prueba basada en propiedades de atomicidad de pesos
    - **Propiedad 11: Atomicidad de configuración de pesos**
    - **Valida: Requisito 2.8**
  - [ ] 4.9 Escribir pruebas unitarias de transformación
    - Cubrir peso omitido/cero, una entidad, precisión numérica, configuración efectiva completa y rechazo sin commit.
    - _Requisitos: 2.3, 2.4, 2.6–2.9_

- [ ] 5. Implementar métricas, explicaciones básicas y recomendación top-K
  - [ ] 5.1 Implementar primitivas de métrica y contribuciones
    - Implementar distancia euclidiana, coseno con control de norma cero y contribuciones por característica usando acumulación compensada.
    - Limitar solo ruido admisible y convertir valores no finitos o fuera de rango en error interno.
    - _Requisitos: 3.2, 3.4, 6.1–6.5_
  - [ ] 5.2 Implementar comparador determinista y top-K acotado
    - Crear comparadores euclidiano/coseno con `EPS_TIE=1e-9`, desempate Unicode ascendente y heap de tamaño K.
    - Excluir la referencia y candidatos coseno de norma cero antes del conteo.
    - _Requisitos: 3.1, 3.3, 3.5, 3.6, 3.9_
  - [ ] 5.3 Implementar el caso de uso de recomendación transaccional
    - Validar referencia, métrica y K; ensamblar valores, posición, contribuciones y configuración efectiva.
    - Publicar solo tras verificar cardinalidad, orden y finitud; conservar el último resultado válido ante cualquier rechazo.
    - _Requisitos: 3.1–3.11, 6.1–6.5, 6.8_
  - [ ] 5.4 Escribir la prueba basada en propiedades de recomendación euclidiana
    - **Propiedad 12: Recomendación euclidiana completa**
    - **Valida: Requisitos 3.1, 3.2, 3.3, 3.6**
  - [ ] 5.5 Escribir la prueba basada en propiedades de recomendación coseno
    - **Propiedad 13: Recomendación coseno completa**
    - **Valida: Requisitos 3.1, 3.4, 3.5, 3.6, 3.9**
  - [ ] 5.6 Escribir la prueba basada en propiedades de valores publicados
    - **Propiedad 22: Recalculabilidad de valores publicados**
    - **Valida: Requisito 6.1**
  - [ ] 5.7 Escribir la prueba basada en propiedades de contribución euclidiana
    - **Propiedad 23: Descomposición de distancia euclidiana**
    - **Valida: Requisitos 6.2, 6.3**
  - [ ] 5.8 Escribir la prueba basada en propiedades de contribución coseno
    - **Propiedad 24: Descomposición de similitud coseno**
    - **Valida: Requisitos 6.4, 6.5**
  - [ ] 5.9 Escribir pruebas unitarias del caso de recomendación
    - Cubrir referencia no apta/cero, K inválido, métrica desconocida, ausencia de candidatos, empates dentro/fuera de tolerancia y conservación del resultado previo.
    - _Requisitos: 3.6–3.11_

- [ ] 6. Construir benchmarks tempranos de factibilidad
  - [ ] 6.1 Implementar el generador reproducible de datasets y el arnés de benchmark nativo
    - Generar matrices row-major de hasta 100.000 × 100 con semilla fija y manifiesto de versión, dimensiones y huella.
    - Incorporar medición local de tiempo por fase y RSS máxima sin observabilidad externa.
    - _Requisitos: 8.1–8.3, 8.8_
  - [ ] 6.2 Implementar benchmarks de validación y recomendación exacta
    - Codificar casos de 100.000 × 100, calentamiento separado y cinco ejecuciones verificadas contra límites de 16 GB y 5 segundos.
    - Hacer que la suite falle automáticamente si una ejecución incumple el umbral.
    - _Requisitos: 8.1, 8.2, 8.8_
  - [ ] 6.3 Implementar un benchmark de riesgo para representante y silueta euclidianos exactos
    - Crear un kernel bloqueado, paralelo y cancelable que mida las fases cuadráticas sobre tamaños escalables y extrapolación claramente etiquetada.
    - Mantener un oráculo exacto para tamaños pequeños; no introducir muestreo ni aproximaciones silenciosas.
    - _Requisitos: 4.8, 4.11–4.16, 8.3, 8.8_
  - [ ] 6.4 Codificar la puerta automatizada de factibilidad
    - Emitir un artefacto local estructurado por fase y un estado de aprobación/fallo que impida dar por validado 8.3 sin el benchmark exacto del Entorno_de_Referencia.
    - Conservar el riesgo explícito para que un incumplimiento fuerce una revisión de requisitos, no una degradación matemática.
    - _Requisitos: 8.3, 8.8_

- [ ] 7. Checkpoint de núcleo inicial y riesgo de rendimiento
  - Asegurar que todas las pruebas y benchmarks automatizados disponibles pasen; preguntar al usuario si surgen dudas o si 8.3 exige revisar requisitos.

- [ ] 8. Implementar clustering determinista Lloyd y esférico
  - [ ] 8.1 Implementar PRNG sembrado e inicialización k-means++
    - Usar ChaCha8, selección sin repetición y fallback por menor identificador cuando todas las disimilitudes sean cero.
    - _Requisitos: 4.17, 8.7_
  - [ ] 8.2 Implementar `lloyd-euclidean-v1`
    - Añadir asignación por distancia cuadrática, actualización por medias, tolerancia, máximo de iteraciones y desempate por `clusterId`.
    - Publicar solo valores euclidianos con raíz cuando el contrato lo requiera.
    - _Requisitos: 4.1–4.6, 4.17, 4.19_
  - [ ] 8.3 Implementar `spherical-kmeans-v1`
    - Normalizar vectores y centros, asignar por coseno y rechazar de forma atómica cualquier entidad ponderada de norma cero.
    - Implementar reparación de centro de suma cero sin producir NaN.
    - _Requisitos: 4.1–4.6, 4.17, 4.19_
  - [ ] 8.4 Implementar reparación determinista de clusters vacíos
    - Procesar vacíos por `clusterId`, seleccionar donante no unitario y mover la entidad peor representada con desempates reglados.
    - Recalcular centros afectados y fallar sin parcial si no se obtienen exactamente K clusters no vacíos.
    - _Requisitos: 4.2–4.6, 4.17_
  - [ ] 8.5 Implementar representantes y agregados de cluster
    - Elegir representantes exactos según media euclidiana/coseno, resolver empates por ID y calcular medias normalizadas y ponderadas con acumulación compensada.
    - Canonizar `clusterId` por menor ID miembro.
    - _Requisitos: 4.7–4.10, 6.6, 6.7_
  - [ ] 8.6 Implementar silueta exacta
    - Calcular `a`, mínimo `b`, singletons, caso `max(a,b)=0`, disimilitud `1-coseno` y media global acotada.
    - Procesar pares por bloques con cancelación y sin publicar resultados parciales.
    - _Requisitos: 4.11–4.16_
  - [ ] 8.7 Implementar el caso de uso y verificador transaccional de clustering
    - Validar K, método/métrica, parámetros, límites y cobertura antes de ejecutar y antes del commit.
    - Asociar configuración efectiva, etiquetar el resultado como agrupación global sin orden y conservar el resultado anterior ante fallo/cancelación.
    - _Requisitos: 4.1–4.6, 4.18, 4.19, 6.6–6.8_
  - [ ] 8.8 Escribir la prueba basada en propiedades de partición
    - Generar casos que provoquen clusters vacíos durante iteraciones.
    - **Propiedad 14: Partición total en K clusters no vacíos**
    - **Valida: Requisitos 4.2, 4.3, 6.6**
  - [ ] 8.9 Escribir la prueba basada en propiedades de atomicidad del clustering
    - **Propiedad 15: Atomicidad de parámetros de clustering**
    - **Valida: Requisito 4.5**
  - [ ] 8.10 Escribir la prueba basada en propiedades del representante
    - **Propiedad 16: Representante óptimo y determinista**
    - **Valida: Requisitos 4.7–4.10**
  - [ ] 8.11 Escribir la prueba basada en propiedades de silueta
    - **Propiedad 17: Silueta exacta, agregada y acotada**
    - **Valida: Requisitos 4.11–4.16**
  - [ ] 8.12 Escribir pruebas unitarias y benchmark completo de clustering
    - Cubrir K/método inválidos, incompatibilidad de métrica, no convergencia, vector/centro cero, reparación múltiple, singletons, cancelación y conservación transaccional.
    - Ampliar la suite nativa con K=2 y K=100, cinco ejecuciones y desglose de inicialización, iteración, reparación, representantes, silueta y serialización.
    - _Requisitos: 4.1–4.19, 8.3, 8.8_

- [ ] 9. Implementar perfiles, filtros y listas curadas
  - [ ] 9.1 Implementar el constructor de perfil de preferencia
    - Validar entidades aptas y distintas; calcular la media componente a componente de vectores normalizados antes de ponderar.
    - _Requisitos: 5.1_
  - [ ] 9.2 Implementar el evaluador de elegibilidad
    - Implementar `eq`, `in`, `gte`, `lte` y `between` sin coerción, con AND de filtros y fallo de elegibilidad ante campo ausente.
    - _Requisitos: 5.4, 5.5_
  - [ ] 9.3 Implementar el caso de uso de lista curada
    - Excluir preferencias, aplicar filtros, excluir candidatos coseno cero, ordenar con el ranker y distinguir todas las causas de lista vacía.
    - Validar K y perfil coseno sin modificar perfil/filtros/resultados vigentes; etiquetar como selección personalizada y adjuntar configuración efectiva.
    - _Requisitos: 5.2–5.12, 6.8_
  - [ ] 9.4 Escribir la prueba basada en propiedades del perfil
    - **Propiedad 18: Perfil como centroide normalizado**
    - **Valida: Requisito 5.1**
  - [ ] 9.5 Escribir la prueba basada en propiedades del conjunto elegible
    - **Propiedad 19: Conjunto elegible de lista curada**
    - **Valida: Requisitos 5.2–5.5**
  - [ ] 9.6 Escribir la prueba basada en propiedades del orden curado
    - **Propiedad 20: Orden total de lista curada**
    - **Valida: Requisitos 5.6–5.8**
  - [ ] 9.7 Escribir la prueba basada en propiedades de atomicidad de K y pruebas de bordes
    - **Propiedad 21: Atomicidad de K en lista curada**
    - Cubrir perfil cero, filtros sin coincidencias, ausencia de candidatos distintos, candidatos coseno cero y etiquetas exclusivas.
    - **Valida: Requisitos 5.9–5.12**

- [ ] 10. Completar explicabilidad, serialización y formato determinista
  - [ ] 10.1 Implementar el modelo exportable versionado
    - Unificar dataset, recomendación, clustering y lista con esquema, configuración efectiva, posiciones y campos aplicables.
    - _Requisitos: 6.8, 7.1–7.6_
  - [ ] 10.2 Implementar serialización e importación de round-trip JSON
    - Emitir RFC 8259 UTF-8 sin BOM y preservar orden, tipos y estados nulo/vacío/ausente.
    - _Requisitos: 7.1, 7.7_
  - [ ] 10.3 Implementar serialización e importación de round-trip CSV
    - Emitir RFC 4180 UTF-8 sin BOM con CRLF, encabezado y escape correcto; aplicar tokens del esquema para estados ausentes.
    - _Requisitos: 7.2, 7.8_
  - [ ] 10.4 Implementar el formateador legible determinista
    - Ordenar campos por esquema y colecciones por posición o ID Unicode; producir bytes estables para entrada y versión idénticas.
    - _Requisitos: 7.3_
  - [ ] 10.5 Implementar exportadores específicos de resultados
    - Preservar posición y valor en recomendaciones/listas; ordenar clustering por ID y preservar asignación y resúmenes explicables.
    - _Requisitos: 6.1, 6.6–6.8, 7.4–7.6_
  - [ ] 10.6 Escribir la prueba basada en propiedades de medias de cluster
    - **Propiedad 25: Medias de cluster verificables**
    - **Valida: Requisito 6.7**
  - [ ] 10.7 Escribir la prueba basada en propiedades de round-trip JSON
    - **Propiedad 26: Round-trip JSON conforme**
    - **Valida: Requisitos 7.1, 7.7**
  - [ ] 10.8 Escribir la prueba basada en propiedades de round-trip CSV
    - Generar comas, comillas, saltos, Unicode y estados de ausencia.
    - **Propiedad 27: Round-trip CSV conforme al esquema**
    - **Valida: Requisitos 7.2, 7.8**
  - [ ] 10.9 Escribir la prueba basada en propiedades del formato legible
    - **Propiedad 28: Formato legible determinista**
    - **Valida: Requisito 7.3**
  - [ ] 10.10 Escribir la prueba basada en propiedades de exportación de resultados
    - **Propiedad 29: Exportación preserva semántica y orden del resultado**
    - **Valida: Requisitos 7.4–7.6**

- [ ] 11. Implementar límites, privacidad de ejecución y reproducibilidad transversal
  - [ ] 11.1 Implementar el validador previo de capacidad
    - Validar N, M, K y cantidad de clusters antes de reservar buffers o invocar algoritmos; informar límites y valores recibidos.
    - Conservar dataset y resultados válidos sin publicar parciales.
    - _Requisitos: 3.10, 4.4, 5.12, 8.5, 8.6_
  - [ ] 11.2 Implementar identidad canónica, semillas y verificación de resultados
    - Calcular huellas locales de dataset/configuración, inyectar RNG y reloj, y verificar orden, valores, asignaciones y configuración antes de commit.
    - _Requisitos: 4.17, 8.7_
  - [ ] 11.3 Escribir la prueba basada en propiedades de límites
    - Usar un doble de algoritmo que demuestre que no fue invocado tras un rechazo.
    - **Propiedad 30: Prevalidación y atomicidad de límites**
    - **Valida: Requisitos 8.5, 8.6**
  - [ ] 11.4 Escribir la prueba basada en propiedades de reproducibilidad
    - Comparar representaciones canónicas de los tres tipos de análisis con dataset, configuración y versión idénticos.
    - **Propiedad 31: Reproducibilidad transversal**
    - **Valida: Requisitos 4.17, 8.7**

- [ ] 12. Integrar WebAssembly, Web Workers e IndexedDB
  - [ ] 12.1 Implementar bindings Wasm de grano grueso
    - Exponer importación, transformación, análisis, paginación y exportación sobre bytes/slices y arreglos tipados, con gestión explícita de memoria.
    - _Requisitos: 8.1–8.4_
  - [ ] 12.2 Implementar el worker coordinador y el protocolo request/response
    - Enrutar uniones discriminadas por `requestId`, transferir buffers, informar progreso por fase y convertir errores sin filtrar datos.
    - _Requisitos: 1.9, 8.1–8.4_
  - [ ] 12.3 Implementar el pool fijo de workers y la cancelación cooperativa
    - Limitar el pool a `min(8, hardwareConcurrency)`, soportar memoria compartida cuando haya aislamiento y fallback homologado como no referente.
    - Consultar un token atómico por bloques y descartar borradores al cancelar o fallar un worker.
    - _Requisitos: 4.6, 8.2–8.4_
  - [ ] 12.4 Implementar `AnalysisGateway` en TypeScript
    - Exponer promesas cancelables para importar, configurar, recomendar, agrupar, curar, paginar, exportar y formatear.
    - Impedir que la UI reciba la matriz completa.
    - _Requisitos: 1.10, 3.1, 4.2, 5.2, 7.1–7.6_
  - [ ] 12.5 Implementar el repositorio IndexedDB transaccional
    - Versionar por sistema, escribir encabezado/páginas/marcador `committed`, ignorar borradores y degradar a memoria ante cuota insuficiente.
    - Persistir configuración y último resultado, pero no el archivo fuente salvo acción explícita.
    - _Requisitos: 2.8, 3.7, 4.4–4.6, 5.12, 8.4, 8.7_
  - [ ] 12.6 Escribir pruebas de integración Rust/Wasm/worker/persistencia
    - Probar contrato binario, transferencia sin clonación, paginación, cancelación, crash/recreación única, commits atómicos y recuperación de IndexedDB.
    - _Requisitos: 3.7–3.11, 4.4–4.6, 5.12, 8.4, 8.7_
  - [ ] 12.7 Escribir smoke tests de memoria y fallback de workers
    - Verificar presupuesto previo de buffers, liberación por fases, modo compartido/fallback y ausencia de resultados parciales ante falta de memoria.
    - _Requisitos: 8.1, 8.5, 8.6_

- [ ] 13. Construir la interfaz React local-first
  - [ ] 13.1 Implementar el shell, enrutamiento local y estado de sesión
    - Crear layout accesible, separación de dataset/configuración/resultados y carga diferida de vistas pesadas sin solicitudes remotas.
    - _Requisitos: 4.18, 5.11, 8.4_
  - [ ] 13.2 Implementar importación, esquema e inspector del dataset
    - Crear `ImportWorkspace` y `DatasetInspector` con selección local, editor de esquema, progreso, informe virtualizado y paginación de aptas/rechazadas.
    - _Requisitos: 1.1–1.10, 8.5, 8.6_
  - [ ] 13.3 Implementar configuración y paneles de los tres análisis
    - Crear `TransformConfigurator`, `RecommendationPanel`, `ClusteringPanel` y `CuratedListPanel` con validación previa y estados independientes.
    - Mantener perfil, filtros, configuración y último resultado válidos ante errores.
    - _Requisitos: 2.6–2.9, 3.7–3.11, 4.1, 4.4–4.6, 5.9–5.12_
  - [ ] 13.4 Implementar vistas de resultados y explicabilidad
    - Mostrar ranking, valores y contribuciones; clustering global sin orden con representante, conteos, medias y silueta; lista como selección personalizada.
    - Consultar páginas y detalles bajo demanda.
    - _Requisitos: 4.18, 5.11, 6.1–6.8_
  - [ ] 13.5 Implementar exportación, monitor de trabajos y controles de sesión
    - Crear `ExportPanel` y `JobMonitor` con descarga por Blob, revocación de URLs, progreso, cancelación, recuperación local y borrado de sesión.
    - _Requisitos: 7.1–7.6, 8.4, 8.7_
  - [ ] 13.6 Escribir pruebas de componentes React
    - Probar habilitación de controles, errores localizados, etiquetas exclusivas, conservación visual, paginación, cancelación y flujos de descarga simulados.
    - _Requisitos: 1.9, 3.7, 4.18, 5.11, 6.8, 7.1–7.6_
  - [ ] 13.7 Escribir pruebas automatizadas de accesibilidad de la interfaz
    - Validar teclado, foco, nombres accesibles, anuncios de progreso/error y tablas/listas virtualizadas sin depender del color.
    - _Requisitos: 1.9, 4.18, 5.11_

- [ ] 14. Endurecer seguridad y comprobar flujos offline
  - [ ] 14.1 Implementar la política local-first y la PWA estática
    - Configurar CSP con `connect-src 'none'`, workers y assets locales, service worker sin sincronización ni caché de archivos fuente y headers de aislamiento para memoria compartida.
    - Añadir guardas de build que rechacen URLs remotas, telemetría y dependencias de runtime no permitidas.
    - _Requisitos: 8.4, 8.8_
  - [ ] 14.2 Escribir E2E offline de los flujos completos
    - Bloquear fetch, XHR, WebSocket, beacon y red tras cargar; importar fixtures, ejecutar los tres análisis, inspeccionar explicaciones, exportar/reimportar, recargar persistencia y cancelar trabajos.
    - Verificar cero solicitudes y conservación transaccional con Playwright.
    - _Requisitos: 1.1–1.10, 2.1–2.9, 3.1–3.11, 4.1–4.19, 5.1–5.12, 6.1–6.8, 7.1–7.8, 8.4, 8.7_
  - [ ] 14.3 Escribir pruebas de privacidad y limpieza local
    - Auditar el bundle por URLs/SDKs prohibidos, comprobar que logs no contienen datos sensibles y verificar borrado de IndexedDB y revocación de Blobs.
    - _Requisitos: 8.4, 8.8_
  - [ ] 14.4 Escribir la matriz automatizada de compatibilidad
    - Ejecutar la suite E2E en Chromium y Firefox soportados, incluyendo fallback sin memoria compartida y detección explícita de entorno no homologado.
    - _Requisitos: 8.4, 8.7, 8.8_
  - [ ] 14.5 Escribir pruebas de exportación descargable byte a byte
    - Validar UTF-8 sin BOM, CRLF CSV, orden Unicode/posición, nombres seguros y round-trip desde los Blobs producidos por la UI.
    - _Requisitos: 7.1–7.8_

- [ ] 15. Completar la validación automatizada de entrega
  - [ ] 15.1 Implementar la suite final de capacidad, rendimiento y memoria
    - Integrar validación 2..100.000 × 2..100, recomendación 100.000 × 100 y clustering K=2..100 con cinco ejecuciones, umbrales y RSS desde solicitud hasta respuesta.
    - Registrar localmente versión, runtime/navegador, semilla, dimensiones, configuración y huella sin infraestructura externa.
    - _Requisitos: 8.1–8.3, 8.8_
  - [ ] 15.2 Implementar el pipeline reproducible de calidad
    - Crear scripts que ejecuten formato, lint, typecheck, builds nativo/Wasm/web, unitarias, las 31 propiedades, integración y E2E sin modo watch.
    - Separar el perfil pesado del Entorno_de_Referencia, pero impedir declarar cumplimiento de rendimiento si no existe un resultado aprobado.
    - _Requisitos: 8.1–8.8_
  - [ ] 15.3 Implementar el manifiesto verificable de criterios de salida
    - Generar desde los resultados de pruebas un manifiesto local que enlace cada requisito con sus suites y marque explícitamente rendimiento no ejecutado o fallido.
    - No permitir que aproximaciones, pruebas omitidas o entornos no homologados aparezcan como cumplimiento.
    - _Requisitos: 1.1–8.8_

- [ ] 16. Checkpoint final
  - Asegurar que todas las pruebas pasen; preguntar al usuario si surgen dudas. Confirmar por separado si los benchmarks 8.1–8.3 fueron ejecutados en el Entorno_de_Referencia y cumplieron sus umbrales.

## Notes

- No se marca ninguna subtarea con `*`: las pruebas unitarias, de integración, E2E, las 31 propiedades y los benchmarks forman parte explícita del alcance solicitado y de los criterios de salida del diseño.
- Cada prueba basada en propiedades debe vivir en un archivo dedicado, usar al menos 100 casos exitosos y conservar el comentario `Feature: motor-recomendacion-similitud, Property N`.
- Los checkpoints no sustituyen validaciones automatizadas ni autorizan aproximaciones. Si el benchmark exacto de clustering incumple 8.3, se debe volver a requisitos antes de continuar con inversiones adicionales o cambiar el algoritmo.
- Las tareas de cada ola pueden ejecutarse en paralelo; una ola comienza únicamente cuando todas las anteriores han terminado.

## Task Dependency Graph

```json
{
  "waves": [
    { "id": 0, "tasks": ["1.1", "1.2"] },
    { "id": 1, "tasks": ["1.3", "1.4", "2.1", "2.2"] },
    { "id": 2, "tasks": ["2.3", "2.4", "3.1", "6.1"] },
    { "id": 3, "tasks": ["3.2", "3.3", "4.1", "5.1", "11.1"] },
    { "id": 4, "tasks": ["3.4", "4.2", "5.2", "11.2"] },
    { "id": 5, "tasks": ["3.5", "3.6", "3.7", "3.8", "3.9", "3.10", "3.11", "4.3", "5.3", "11.3"] },
    { "id": 6, "tasks": ["4.4", "4.5", "4.6", "4.7", "4.8", "4.9", "5.4", "5.5", "5.6", "5.7", "5.8", "5.9", "6.2"] },
    { "id": 7, "tasks": ["6.3"] },
    { "id": 8, "tasks": ["6.4"] },
    { "id": 9, "tasks": ["8.1", "9.1", "9.2", "10.1", "12.1"] },
    { "id": 10, "tasks": ["8.2", "8.3", "10.2", "10.3", "10.4", "12.2", "12.5"] },
    { "id": 11, "tasks": ["8.4", "9.3", "10.5", "12.3"] },
    { "id": 12, "tasks": ["8.5", "8.6", "9.4", "9.5", "9.6", "9.7", "10.7", "10.8", "10.9", "10.10", "12.4"] },
    { "id": 13, "tasks": ["8.7", "10.6", "12.7"] },
    { "id": 14, "tasks": ["8.8", "8.9", "8.10", "8.11", "8.12", "11.4", "12.6"] },
    { "id": 15, "tasks": ["13.1", "13.2", "13.3"] },
    { "id": 16, "tasks": ["13.4", "13.5", "13.6", "13.7", "14.1"] },
    { "id": 17, "tasks": ["14.2", "14.3", "14.4", "14.5"] },
    { "id": 18, "tasks": ["15.1", "15.2"] },
    { "id": 19, "tasks": ["15.3"] }
  ]
}
```
