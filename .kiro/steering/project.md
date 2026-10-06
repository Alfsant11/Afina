# Steering del proyecto Afina

## Propósito

Afina es una aplicación web analítica local-first para recomendar y agrupar entidades multidimensionales —como canciones, películas o ítems de juego— a partir de características numéricas normalizadas y ponderadas.

El producto debe ofrecer tres resultados claramente diferenciados:

- recomendaciones ordenadas respecto de una entidad de referencia;
- clusters globales de afinidad, sin significado de ranking;
- listas curadas ordenadas respecto de un perfil construido con preferencias del usuario.

La especificación vigente de `motor-recomendacion-similitud` es la fuente de verdad. Antes de implementar o modificar comportamiento, consulta `requirements.md`, `design.md` y `tasks.md` de esa especificación. No cambies una decisión funcional importante solo en el código: actualiza primero la especificación correspondiente.

## Idioma y nomenclatura

- Redacta documentación, mensajes visibles y explicaciones para el usuario en español.
- Usa inglés para nombres de archivos, módulos, tipos, funciones, variables, eventos y códigos de error.
- Conserva los términos funcionales de la especificación al explicar el dominio.
- Usa códigos de error estables y legibles por máquinas; localiza el texto visible en la capa de presentación.

## Arquitectura obligatoria

- Frontend: React + TypeScript estricto + Vite, bajo `apps/web`.
- Núcleo analítico: Rust con `f64`, compilable como biblioteca nativa y como WebAssembly.
- Separa los crates y módulos de dominio matemático, importación/exportación, casos de uso, bindings Wasm y benchmarks.
- Ejecuta cálculos costosos fuera del hilo principal mediante Web Workers.
- Usa mensajes tipados y buffers transferibles o compartidos; no cruces objetos por entidad durante operaciones masivas.
- Usa IndexedDB solo para configuración, esquema y último resultado válido. No persistas el archivo fuente sin una acción explícita.
- El dominio Rust debe ser puro: sin DOM, red, IndexedDB, reloj global ni aleatoriedad global.
- Inyecta semilla, reloj y cancelación a través de interfaces explícitas.
- Mantén las dependencias apuntando hacia el dominio; la UI no debe implementar fórmulas ni reglas analíticas.

## Privacidad y ejecución local

- Todo dataset y resultado debe permanecer en el entorno configurado por el usuario.
- No añadas telemetría, analítica, sincronización, clientes HTTP, CDN ni dependencias de servicios externos en runtime.
- La aplicación debe funcionar offline después de cargar sus assets locales.
- Configura una CSP restrictiva con `connect-src 'none'` y audita el bundle para detectar URLs o SDK remotos.
- No registres filas, vectores completos, metadatos sensibles ni contenido del archivo del usuario.
- Libera URLs de Blob y ofrece borrado explícito del estado persistido en IndexedDB.

## Reglas del dominio matemático

- Acepta únicamente números reales finitos; rechaza `NaN`, infinitos, booleanos y valores numéricos ausentes.
- Almacena matrices numéricas contiguas en orden row-major.
- Calcula parámetros de normalización únicamente sobre entidades aptas.
- Min-max: `(x - min) / (max - min)`.
- Z-score: `(x - mean) / populationStdDev`, usando divisor `N`.
- Una característica constante se normaliza a cero.
- Aplica pesos después de normalizar. Un peso ausente vale `1`; un peso `0` es válido; cualquier peso negativo o no finito invalida la configuración completa.
- Distancia euclidiana: `sqrt(sum((x_j - y_j)^2))`.
- Similitud coseno: `dot(x, y) / (norm(x) * norm(y))`, solo para vectores con norma distinta de cero.
- Usa `EPS_TIE = 1e-9` para empates analíticos y `1e-6` para validar media/desviación de Z-score.
- Ante empate, ordena por identificador Unicode ascendente; en asignación de clusters usa después el `clusterId` estable cuando corresponda.
- Usa acumulación compensada para sumas que afecten valores publicados.
- Nunca conviertas silenciosamente resultados no finitos o errores matemáticos en valores válidos.

## Tratamiento de vectores cero

- Euclidiana admite vectores cero.
- En recomendación coseno, rechaza una referencia de norma cero y excluye candidatos de norma cero antes de contar resultados.
- En clustering coseno, cualquier entidad ponderada de norma cero invalida la solicitud completa.
- En listas curadas con coseno, rechaza un perfil de norma cero y excluye candidatos de norma cero.
- No publiques `NaN`, infinito ni resultados parciales para estos casos.

## Clustering y determinismo

- Admite `lloyd-euclidean-v1` únicamente con distancia euclidiana.
- Admite `spherical-kmeans-v1` únicamente con similitud coseno.
- Usa inicialización k-means++ y PRNG ChaCha8 sembrado; nunca uses `Math.random` ni RNG global.
- Repara clusters vacíos de forma determinista moviendo la entidad peor representada desde un cluster donante no unitario.
- Produce exactamente `K` clusters no vacíos o rechaza toda la ejecución.
- Canoniza los identificadores de cluster para que el resultado sea reproducible.
- Calcula representantes y silueta exactamente según la especificación.
- No introduzcas muestreo ni aproximaciones silenciosas. Si el cálculo exacto no cumple el objetivo de rendimiento, detén el trabajo afectado y plantea un cambio explícito de requisitos.

## Importación, validación y exportación

- Soporta JSON conforme a RFC 8259 y CSV conforme a RFC 4180.
- Procesa entradas de forma incremental y limita los detalles de error sin perder los totales.
- Recorta los identificadores; deben tener entre 1 y 255 caracteres y ser únicos tras el recorte.
- Si un identificador está duplicado, rechaza todas sus apariciones.
- Un error de entidad excluye la entidad completa; un error global o cero entidades válidas produce un dataset analítico vacío.
- Conserva de forma explícita la diferencia entre campo ausente, nulo y cadena vacía.
- Exporta JSON y CSV en UTF-8 sin BOM; usa CRLF para CSV.
- Las serializaciones deben ser deterministas y permitir round-trip conforme al esquema declarado.
- Ordena recomendaciones y listas por posición; ordena agrupaciones por identificador de entidad.

## Estado, errores y transacciones

- Trata datasets, configuraciones y solicitudes como valores inmutables.
- Ejecuta cada análisis en un borrador aislado.
- Sustituye el último resultado válido solo después de verificar cardinalidad, cobertura, orden, finitud y configuración efectiva.
- Error, cancelación, crash de worker o falta de memoria deben descartar el borrador y conservar el último estado válido.
- Todos los errores públicos deben incluir código, categoría y detalles mínimos seguros.
- Valida límites antes de reservar buffers grandes o invocar algoritmos.
- La cancelación debe ser cooperativa y comprobarse entre bloques de trabajo.

## Rendimiento

- Prioriza buffers contiguos, procesamiento por bloques, vistas ponderadas y top-K con heap acotado.
- Evita copiar la matriz entre TypeScript, Wasm y workers.
- Mantén la UI paginada o virtualizada; nunca envíes la matriz completa al hilo de presentación.
- Ejecuta benchmarks nativos tempranos antes de construir toda la interfaz.
- Mide tiempo extremo a extremo y memoria residente máxima en el entorno de referencia, sin carga concurrente.
- El representante euclidiano y la silueta exacta pueden ser cuadráticos. Conserva este riesgo visible y no declares cumplimiento sin benchmark real.

## Calidad y pruebas

- Todo comportamiento nuevo debe incluir pruebas en la misma tarea.
- Rust: pruebas unitarias, integración y property-based testing con `proptest`.
- Web: pruebas unitarias/de componentes y flujos E2E offline con Playwright.
- Cada propiedad del diseño debe implementarse en un archivo dedicado, con semilla reproducible y al menos 100 casos exitosos.
- Incluye oráculos independientes para fórmulas, orden, cardinalidad, round-trip y silueta.
- Cubre explícitamente límites, Unicode, empates, columnas constantes, vectores cero, clusters vacíos, cancelación y conservación transaccional.
- Los tests y builds ejecutados por agentes deben terminar; nunca uses modo watch ni servidores persistentes.
- Antes de completar una tarea, ejecuta las pruebas dirigidas, formato, lint, typecheck y build de los paquetes afectados.
- No declares aprobados los objetivos de rendimiento si los benchmarks pesados no se ejecutaron en el entorno de referencia.

## Convenciones de implementación

- Mantén TypeScript en modo estricto y evita `any`; valida datos desconocidos en los límites.
- En Rust, representa invariantes con tipos y constructores validados; evita `unwrap`, `expect` y `panic!` en rutas de producción.
- Prefiere funciones pequeñas y puras en el dominio y adaptadores explícitos para efectos secundarios.
- Fija versiones exactas al añadir dependencias y justifica cada dependencia nueva.
- No dupliques reglas entre TypeScript y Rust: Rust es la autoridad analítica; TypeScript valida para UX, pero el núcleo vuelve a validar.
- Documenta complejidad temporal y espacial en algoritmos no triviales.
- Conserva compatibilidad entre contratos TypeScript/Wasm mediante una versión explícita.
- No mezcles refactors no relacionados con la tarea activa.

## Ejecución del plan

- Sigue el orden y las dependencias de `tasks.md`.
- Implementa una subtarea concreta por vez y mantén sus referencias a requisitos.
- Respeta los checkpoints, especialmente la puerta de factibilidad de rendimiento previa a la inversión completa en UI.
- Si código, requisitos y diseño discrepan, detente y corrige primero el documento fuente de verdad apropiado.
- Una tarea solo se considera terminada cuando su código, pruebas y validaciones aplicables pasan.