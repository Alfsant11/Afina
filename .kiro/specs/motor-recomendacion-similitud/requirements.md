# Requirements Document

## Introduction

Este documento define una funcionalidad que analiza colecciones de canciones, películas, ítems de juego u otras entidades descritas mediante valores numéricos. La funcionalidad permite comparar una entidad concreta con otras, dividir una colección completa en grupos afines y generar listas ordenadas a partir de preferencias seleccionadas. Los resultados pueden inspeccionarse y exportarse.

Cada colección comparte un mismo conjunto de características numéricas, cada entidad posee un identificador único después de eliminar espacios iniciales y finales, y la primera versión procesa una colección por ejecución. La recomendación por entidad, la agrupación por afinidad y la lista curada son resultados distintos: la recomendación produce vecinos ordenados respecto de una referencia, la agrupación asigna todas las entidades aptas a grupos sin generar una clasificación personalizada y la lista curada ordena candidatos respecto de un perfil de preferencia.

## Glossary

- **Sistema_de_Recomendación**: Funcionalidad completa que importa, valida, transforma, analiza y exporta colecciones de entidades multidimensionales.
- **Entidad**: Elemento individual que puede representar una canción, película, ítem de juego u otro objeto analizable.
- **Identificador_de_Entidad**: Valor textual de 1 a 255 caracteres, medidos después de eliminar espacios iniciales y finales, que distingue una Entidad dentro de un Conjunto_de_Datos.
- **Característica_Numérica**: Atributo cuantitativo real y finito compartido por las Entidades de un Conjunto_de_Datos.
- **Metadato**: Atributo descriptivo no utilizado en cálculos matemáticos, como título, categoría o dirección de recurso.
- **Esquema_Declarado**: Definición de los campos, nombres, orden, tipos, obligatoriedad y representación de valores nulos, vacíos o ausentes de un Conjunto_de_Datos.
- **Registro**: Unidad estructurada de entrada o salida que representa una Entidad o un elemento de un resultado analítico.
- **Error_de_Entidad**: Incumplimiento asociado a uno o más campos de una Entidad concreta.
- **Error_Global**: Incumplimiento del archivo, del formato, del Esquema_Declarado o de los encabezados que impide interpretar de manera fiable los Registros.
- **Conjunto_de_Datos**: Colección de Entidades con un Esquema_Declarado común de Características_Numéricas y Metadatos opcionales.
- **Entidad_Apta**: Entidad que supera la validación completa y dispone de los valores requeridos para el análisis seleccionado.
- **Vector_de_Características**: Secuencia ordenada de valores de las Características_Numéricas de una Entidad.
- **Vector_Normalizado**: Vector_de_Características cuyos componentes resultan de aplicar el método de normalización seleccionado.
- **Vector_Ponderado**: Vector cuyos componentes resultan de multiplicar cada componente del Vector_Normalizado por el Peso_de_Característica correspondiente.
- **Normalizador**: Componente del Sistema_de_Recomendación que transforma Características_Numéricas a escalas comparables y calcula Vectores_Ponderados.
- **Normalización_Min_Max**: Transformación calculada sobre las Entidades_Aptas mediante la expresión `(x - mínimo) / (máximo - mínimo)` para cada Característica_Numérica no constante.
- **Estandarización_Z**: Transformación calculada sobre las Entidades_Aptas mediante la expresión `(x - media) / desviación_poblacional`, donde la desviación poblacional usa divisor `N`.
- **Parámetros_de_Transformación**: Mínimos y máximos de la Normalización_Min_Max, o medias y desviaciones poblacionales de la Estandarización_Z.
- **Peso_de_Característica**: Número real finito mayor o igual que cero que controla la influencia relativa de una Característica_Numérica.
- **Métrica_de_Similitud**: Regla seleccionada para comparar dos Vectores_Ponderados; los valores admitidos son Distancia_Euclidiana y Similitud_Coseno.
- **Distancia_Euclidiana**: Métrica_de_Similitud en la que un valor menor representa mayor afinidad.
- **Similitud_Coseno**: Métrica_de_Similitud con valores entre -1 y 1, en la que un valor mayor representa mayor afinidad y cuyo cálculo requiere vectores de norma distinta de cero.
- **Norma**: Raíz cuadrada de la suma de los cuadrados de los componentes de un vector.
- **Recomendación_por_Entidad**: Resultado ordenado de Entidades vecinas calculado respecto de una única Entidad_de_Referencia.
- **Entidad_de_Referencia**: Entidad cuyo Vector_Ponderado se usa como punto de comparación para una Recomendación_por_Entidad.
- **Candidato_de_Recomendación**: Entidad_Apta distinta de la Entidad_de_Referencia y, para Similitud_Coseno, con Vector_Ponderado de norma distinta de cero.
- **Cantidad_K**: Número entero que limita la cantidad solicitada de Entidades en una Recomendación_por_Entidad o una Lista_Curada.
- **Cluster_de_Afinidad**: Grupo no vacío de Entidades_Aptas que comparten proximidad multidimensional según la configuración de agrupación.
- **Agrupación_de_Afinidad**: Resultado sin orden global que asigna cada Entidad_Apta a exactamente un Cluster_de_Afinidad.
- **Cantidad_de_Clusters**: Número entero solicitado de Clusters_de_Afinidad.
- **Método_de_Agrupación**: Procedimiento de agrupación identificado por nombre y declarado como disponible por el Sistema_de_Recomendación.
- **Configuración_de_Agrupación**: Cantidad_de_Clusters, Método_de_Agrupación, parámetros del método, normalización, Parámetros_de_Transformación, Pesos_de_Característica, Métrica_de_Similitud y Semilla_de_Ejecución efectivos.
- **Representante_de_Cluster**: Entidad del Cluster_de_Afinidad con la mayor afinidad promedio respecto de las demás Entidades del mismo Cluster_de_Afinidad según la Métrica_de_Similitud seleccionada.
- **Coeficiente_de_Silueta**: Media de los coeficientes individuales de separación y cohesión de una Agrupación_de_Afinidad, comprendida entre -1 y 1.
- **Perfil_de_Preferencia**: Vector derivado de una o más Entidades_Aptas distintas seleccionadas por el Usuario para generar una Lista_Curada.
- **Lista_Curada**: Resultado personalizado y ordenado de Entidades calculado respecto de un Perfil_de_Preferencia.
- **Filtro_de_Elegibilidad**: Regla explícita sobre Metadatos o rangos numéricos que determina qué Entidades pueden aparecer en una Lista_Curada.
- **Contribución_de_Característica**: Valor que expresa la participación verificable de una Característica_Numérica en la comparación de dos Vectores_Ponderados.
- **Importador**: Componente que interpreta una representación JSON o CSV y produce un Conjunto_de_Datos.
- **Serializador**: Componente que convierte un Conjunto_de_Datos o un resultado analítico a JSON o CSV válido.
- **Formateador_Legible**: Componente que genera una representación textual determinista y legible de un Conjunto_de_Datos o resultado analítico.
- **JSON**: Formato de intercambio definido por RFC 8259.
- **CSV**: Formato tabular definido por RFC 4180, con una fila de encabezados.
- **Informe_de_Validación**: Colección estructurada de errores y totales que identifica archivo, Registro, campo y causa de cada dato rechazado.
- **Semilla_de_Ejecución**: Número entero que permite reproducir operaciones con decisiones pseudoaleatorias.
- **Configuración_Efectiva**: Valores validados de normalización, Parámetros_de_Transformación, Pesos_de_Característica, Métrica_de_Similitud, parámetros analíticos y Semilla_de_Ejecución usados para producir un resultado.
- **Usuario**: Persona que configura y ejecuta el Sistema_de_Recomendación.
- **Entorno_de_Referencia**: Equipo con 8 núcleos de procesador, 16 GB de memoria disponible y almacenamiento de estado sólido, sin aceleración por procesador gráfico ni carga concurrente.
- **Memoria_Residente_Máxima**: Mayor cantidad de memoria física residente atribuida al proceso del Sistema_de_Recomendación desde la recepción de una solicitud hasta la entrega del resultado o error.
- **Tiempo_Extremo_a_Extremo**: Tiempo transcurrido desde la recepción completa de una solicitud hasta la entrega completa del resultado o error.
- **Versión_del_Sistema**: Identificador inmutable de la versión ejecutada del Sistema_de_Recomendación.

## Requirements

### Requisito 1: Importación y validación del conjunto de datos

**Historia de usuario:** Como analista, quiero cargar entidades con características numéricas y metadatos, para preparar una colección fiable para el análisis.

#### Criterios de aceptación

1. CUANDO el Usuario proporcione un archivo JSON válido conforme a RFC 8259 y al Esquema_Declarado, EL Importador DEBERÁ producir exactamente una Entidad por cada Registro válido.
2. CUANDO el Usuario proporcione un archivo CSV válido conforme a RFC 4180 y al Esquema_Declarado, EL Importador DEBERÁ exigir encabezados únicos y no vacíos después de eliminar espacios iniciales y finales, al menos una columna declarada como Característica_Numérica y exactamente una Entidad por cada fila no vacía.
3. CUANDO el Sistema_de_Recomendación valide un Identificador_de_Entidad, EL Sistema_de_Recomendación DEBERÁ eliminar los espacios iniciales y finales y aceptar únicamente un valor resultante de 1 a 255 caracteres.
4. CUANDO el Sistema_de_Recomendación valide los Identificadores_de_Entidad, EL Sistema_de_Recomendación DEBERÁ evaluar la unicidad sobre los valores resultantes de eliminar los espacios iniciales y finales.
5. SI dos o más Entidades tienen el mismo Identificador_de_Entidad después de eliminar los espacios iniciales y finales, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar todas las Entidades que compartan el Identificador_de_Entidad duplicado.
6. CUANDO el Sistema_de_Recomendación valide una Característica_Numérica, EL Sistema_de_Recomendación DEBERÁ aceptar únicamente un número real finito y clasificar como error los valores vacíos, nulos, booleanos, no numéricos, `NaN` o infinitos.
7. SI una Entidad contiene un Error_de_Entidad, ENTONCES EL Sistema_de_Recomendación DEBERÁ excluir la Entidad completa del Conjunto_de_Datos apto para análisis.
8. SI la importación contiene un Error_Global o no contiene ninguna Entidad válida, ENTONCES EL Importador DEBERÁ producir un Conjunto_de_Datos vacío sin Entidades_Aptas.
9. CUANDO finalice la validación, EL Sistema_de_Recomendación DEBERÁ producir un Informe_de_Validación que identifique el archivo, el Registro, el campo y la causa de cada rechazo y contabilice las Entidades aceptadas y rechazadas.
10. MIENTRAS un Informe_de_Validación contenga errores, EL Sistema_de_Recomendación DEBERÁ permitir el análisis únicamente de las Entidades_Aptas incluidas en el Conjunto_de_Datos resultante.

### Requisito 2: Normalización y ponderación de características

**Historia de usuario:** Como analista, quiero llevar las características a escalas comparables y ajustar su importancia, para evitar que las unidades originales distorsionen la afinidad.

#### Criterios de aceptación

1. CUANDO el Usuario seleccione Normalización_Min_Max, EL Normalizador DEBERÁ calcular para cada Característica_Numérica no constante y cada Entidad_Apta el valor `(x - mínimo) / (máximo - mínimo)`, usando el mínimo y el máximo observados entre todas las Entidades_Aptas.
2. CUANDO el Usuario seleccione Estandarización_Z, EL Normalizador DEBERÁ calcular para cada Característica_Numérica no constante y cada Entidad_Apta el valor `(x - media) / desviación_poblacional`, usando una varianza poblacional con divisor igual a la cantidad `N` de Entidades_Aptas.
3. CUANDO finalice la Estandarización_Z de una Característica_Numérica no constante, EL Normalizador DEBERÁ producir valores con media 0 y desviación poblacional 1 dentro de una tolerancia absoluta de 0,000001.
4. SI una Característica_Numérica tiene varianza cero, incluido el caso `N = 1`, ENTONCES EL Normalizador DEBERÁ asignar el valor normalizado 0 a esa Característica_Numérica para todas las Entidades_Aptas.
5. CUANDO el Usuario asigne un Peso_de_Característica válido, EL Normalizador DEBERÁ calcular cada componente del Vector_Ponderado mediante la expresión `valor_ponderado = valor_normalizado × peso`.
6. SI el Usuario omite el Peso_de_Característica de una Característica_Numérica, ENTONCES EL Normalizador DEBERÁ usar el peso predeterminado 1 para esa Característica_Numérica.
7. CUANDO una Característica_Numérica tenga Peso_de_Característica 0, EL Normalizador DEBERÁ producir el componente 0 en todos los Vectores_Ponderados sin modificar los valores conservados en los Vectores_Normalizados.
8. SI cualquier Peso_de_Característica es negativo, no numérico o no finito, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar la configuración completa, identificar la Característica_Numérica afectada y conservar la última configuración válida.
9. CUANDO finalice una normalización válida, EL Sistema_de_Recomendación DEBERÁ asociar al resultado analítico el método de normalización, los Parámetros_de_Transformación y todos los Pesos_de_Característica efectivos.

### Requisito 3: Recomendaciones por entidad

**Historia de usuario:** Como Usuario, quiero encontrar las entidades más parecidas a una entidad concreta, para descubrir vecinos relevantes de una referencia conocida.

#### Criterios de aceptación

1. CUANDO el Usuario solicite una Recomendación_por_Entidad válida, EL Sistema_de_Recomendación DEBERÁ incluir exactamente `mínimo(Cantidad_K, cantidad de Candidatos_de_Recomendación)` Entidades.
2. CUANDO la Métrica_de_Similitud sea Distancia_Euclidiana, EL Sistema_de_Recomendación DEBERÁ calcular `d(x,y) = raíz(sumatoria desde j=1 hasta m de (x_j - y_j)²)` sobre los Vectores_Ponderados derivados de los Vectores_Normalizados de la Entidad_de_Referencia y cada Candidato_de_Recomendación.
3. CUANDO la Métrica_de_Similitud sea Distancia_Euclidiana, EL Sistema_de_Recomendación DEBERÁ ordenar la Recomendación_por_Entidad de menor a mayor distancia.
4. CUANDO la Métrica_de_Similitud sea Similitud_Coseno, EL Sistema_de_Recomendación DEBERÁ calcular `s(x,y) = sumatoria desde j=1 hasta m de (x_j × y_j) / (Norma(x) × Norma(y))` sobre los Vectores_Ponderados derivados de los Vectores_Normalizados de la Entidad_de_Referencia y cada Candidato_de_Recomendación.
5. CUANDO la Métrica_de_Similitud sea Similitud_Coseno, EL Sistema_de_Recomendación DEBERÁ ordenar la Recomendación_por_Entidad de mayor a menor similitud.
6. CUANDO dos Entidades obtengan valores de comparación cuya diferencia absoluta sea menor o igual que 0,000000001, EL Sistema_de_Recomendación DEBERÁ desempatar las Entidades mediante orden ascendente del Identificador_de_Entidad.
7. SI la Entidad_de_Referencia no es una Entidad_Apta, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar la solicitud, identificar la causa y conservar cualquier Recomendación_por_Entidad válida anterior sin sustitución.
8. SI la Métrica_de_Similitud es Similitud_Coseno y el Vector_Ponderado de la Entidad_de_Referencia tiene Norma cero, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar la solicitud y conservar cualquier Recomendación_por_Entidad válida anterior sin sustitución.
9. MIENTRAS la Métrica_de_Similitud sea Similitud_Coseno, EL Sistema_de_Recomendación DEBERÁ excluir de la Recomendación_por_Entidad y del conteo de Candidatos_de_Recomendación toda Entidad_Apta cuyo Vector_Ponderado tenga Norma cero.
10. SI la Cantidad_K no es un número entero entre 1 y la cantidad total de Entidades del Conjunto_de_Datos, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar la solicitud con un error de rango.
11. SI la Métrica_de_Similitud no es Distancia_Euclidiana ni Similitud_Coseno, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar la solicitud e identificar el valor no admitido.

### Requisito 4: Clusters de afinidad

**Historia de usuario:** Como analista, quiero dividir el conjunto completo en grupos de afinidad, para explorar estructuras globales distintas de las recomendaciones respecto de una sola entidad.

#### Criterios de aceptación

1. CUANDO el Usuario solicite una Agrupación_de_Afinidad, EL Sistema_de_Recomendación DEBERÁ exigir una Cantidad_de_Clusters y un Método_de_Agrupación declarado como disponible.
2. CUANDO el Sistema_de_Recomendación produzca una Agrupación_de_Afinidad válida, EL Sistema_de_Recomendación DEBERÁ asignar cada Entidad_Apta a exactamente un Cluster_de_Afinidad.
3. CUANDO el Sistema_de_Recomendación produzca una Agrupación_de_Afinidad válida, EL Sistema_de_Recomendación DEBERÁ producir exactamente la Cantidad_de_Clusters solicitada con al menos una Entidad_Apta en cada Cluster_de_Afinidad.
4. SI la Cantidad_de_Clusters no es un número entero entre 2 y la cantidad de Entidades_Aptas, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar la solicitud sin producir un resultado parcial y conservar cualquier Agrupación_de_Afinidad válida anterior.
5. SI el Método_de_Agrupación no está declarado como disponible o sus parámetros no son válidos, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar la solicitud sin producir un resultado parcial y conservar cualquier Agrupación_de_Afinidad válida anterior.
6. SI el Método_de_Agrupación no produce exactamente la Cantidad_de_Clusters solicitada con al menos una Entidad_Apta por Cluster_de_Afinidad, ENTONCES EL Sistema_de_Recomendación DEBERÁ declarar fallida la agrupación, descartar el resultado parcial y conservar cualquier Agrupación_de_Afinidad válida anterior.
7. CUANDO un Cluster_de_Afinidad contenga una sola Entidad_Apta, EL Sistema_de_Recomendación DEBERÁ seleccionar esa Entidad_Apta como Representante_de_Cluster.
8. CUANDO la Métrica_de_Similitud sea Distancia_Euclidiana y un Cluster_de_Afinidad contenga más de una Entidad_Apta, EL Sistema_de_Recomendación DEBERÁ seleccionar como Representante_de_Cluster la Entidad con menor distancia media respecto de las demás Entidades del Cluster_de_Afinidad.
9. CUANDO la Métrica_de_Similitud sea Similitud_Coseno y un Cluster_de_Afinidad contenga más de una Entidad_Apta, EL Sistema_de_Recomendación DEBERÁ seleccionar como Representante_de_Cluster la Entidad con mayor similitud media respecto de las demás Entidades del Cluster_de_Afinidad.
10. CUANDO dos candidatas a Representante_de_Cluster tengan promedios cuya diferencia absoluta sea menor o igual que 0,000000001, EL Sistema_de_Recomendación DEBERÁ seleccionar la Entidad con el Identificador_de_Entidad menor en orden ascendente.
11. CUANDO el Sistema_de_Recomendación calcule el coeficiente individual de una Entidad_Apta perteneciente a un Cluster_de_Afinidad no unitario, EL Sistema_de_Recomendación DEBERÁ usar `(b - a) / máximo(a,b)`, donde `a` es la disimilitud media respecto de las demás Entidades del propio Cluster_de_Afinidad y `b` es la menor disimilitud media respecto de todas las Entidades de cualquier otro Cluster_de_Afinidad.
12. SI `máximo(a,b)` es igual a 0, ENTONCES EL Sistema_de_Recomendación DEBERÁ asignar 0 como coeficiente individual de silueta.
13. CUANDO el Sistema_de_Recomendación calcule disimilitudes para el Coeficiente_de_Silueta, EL Sistema_de_Recomendación DEBERÁ usar la Distancia_Euclidiana o el valor `1 - Similitud_Coseno`, según la Métrica_de_Similitud seleccionada.
14. CUANDO una Entidad_Apta pertenezca a un Cluster_de_Afinidad unitario, EL Sistema_de_Recomendación DEBERÁ asignar 0 como coeficiente individual de silueta.
15. CUANDO finalice una Agrupación_de_Afinidad, EL Sistema_de_Recomendación DEBERÁ calcular el Coeficiente_de_Silueta global como la media aritmética de los coeficientes individuales de todas las Entidades_Aptas.
16. CUANDO el Sistema_de_Recomendación produzca un Coeficiente_de_Silueta global, EL Sistema_de_Recomendación DEBERÁ entregar un valor dentro del intervalo de -1 a 1 con una tolerancia absoluta de 0,000000001.
17. CUANDO el Usuario repita una Agrupación_de_Afinidad con el mismo Conjunto_de_Datos y la misma Configuración_de_Agrupación, EL Sistema_de_Recomendación DEBERÁ producir las mismas asignaciones de Identificadores_de_Entidad.
18. MIENTRAS el Sistema_de_Recomendación presente una Agrupación_de_Afinidad, EL Sistema_de_Recomendación DEBERÁ etiquetar el resultado exclusivamente como agrupación global sin orden.
19. CUANDO finalice una Agrupación_de_Afinidad válida, EL Sistema_de_Recomendación DEBERÁ asociar la Configuración_de_Agrupación efectiva al resultado.

### Requisito 5: Listas curadas personalizadas

**Historia de usuario:** Como Usuario, quiero construir una lista a partir de varias entidades preferidas y filtros, para obtener una selección personalizada y exportable.

#### Criterios de aceptación

1. CUANDO el Usuario seleccione una o más Entidades_Aptas distintas como preferencias, EL Sistema_de_Recomendación DEBERÁ derivar el Perfil_de_Preferencia mediante la media aritmética, componente por componente, de los Vectores_Normalizados seleccionados.
2. CUANDO el Usuario solicite una Lista_Curada válida, EL Sistema_de_Recomendación DEBERÁ incluir exactamente `mínimo(Cantidad_K, cantidad de candidatos restantes después de exclusiones y Filtros_de_Elegibilidad)` Entidades_Aptas.
3. CUANDO el Sistema_de_Recomendación determine los candidatos de una Lista_Curada, EL Sistema_de_Recomendación DEBERÁ excluir las Entidades usadas para formar el Perfil_de_Preferencia antes de calcular la cantidad y el orden del resultado.
4. DONDE el Usuario configure uno o más Filtros_de_Elegibilidad, EL Sistema_de_Recomendación DEBERÁ considerar elegible únicamente una Entidad que satisfaga conjuntamente todos los Filtros_de_Elegibilidad configurados.
5. SI una Entidad carece de un campo requerido por un Filtro_de_Elegibilidad, ENTONCES EL Sistema_de_Recomendación DEBERÁ considerar que la Entidad no satisface ese Filtro_de_Elegibilidad.
6. CUANDO la Métrica_de_Similitud de una Lista_Curada sea Distancia_Euclidiana, EL Sistema_de_Recomendación DEBERÁ ordenar la Lista_Curada de menor a mayor distancia respecto del Perfil_de_Preferencia.
7. CUANDO la Métrica_de_Similitud de una Lista_Curada sea Similitud_Coseno, EL Sistema_de_Recomendación DEBERÁ ordenar la Lista_Curada de mayor a menor similitud respecto del Perfil_de_Preferencia.
8. CUANDO dos candidatas obtengan valores de comparación cuya diferencia absoluta sea menor o igual que 0,000000001, EL Sistema_de_Recomendación DEBERÁ desempatar las Entidades mediante orden ascendente del Identificador_de_Entidad.
9. SI una Lista_Curada queda vacía porque ningún candidato satisface los Filtros_de_Elegibilidad, ENTONCES EL Sistema_de_Recomendación DEBERÁ devolver una Lista_Curada vacía con la causa de exclusión por filtros registrada.
10. SI una Lista_Curada queda vacía porque no existen Entidades_Aptas distintas de las preferencias seleccionadas, ENTONCES EL Sistema_de_Recomendación DEBERÁ devolver una Lista_Curada vacía con la causa de ausencia de candidatos distintos registrada.
11. MIENTRAS el Sistema_de_Recomendación presente una Lista_Curada, EL Sistema_de_Recomendación DEBERÁ etiquetar el resultado exclusivamente como selección personalizada.
12. SI la Cantidad_K no es un número entero mayor o igual que 1, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar la solicitud sin eliminar ni modificar el Perfil_de_Preferencia ni los Filtros_de_Elegibilidad vigentes.

### Requisito 6: Explicación y trazabilidad de resultados

**Historia de usuario:** Como analista, quiero comprender por qué las entidades aparecen juntas o se recomiendan, para evaluar la utilidad del resultado.

#### Criterios de aceptación

1. CUANDO el Sistema_de_Recomendación produzca una Recomendación_por_Entidad, EL Sistema_de_Recomendación DEBERÁ incluir para cada Entidad recomendada el valor de la Métrica_de_Similitud verificable mediante recálculo con una tolerancia absoluta de 0,000000001.
2. CUANDO la Métrica_de_Similitud sea Distancia_Euclidiana, EL Sistema_de_Recomendación DEBERÁ calcular para cada Característica_Numérica la Contribución_de_Característica `c_j = (x_j - y_j)²` sobre los componentes de los Vectores_Ponderados comparados.
3. CUANDO la Métrica_de_Similitud sea Distancia_Euclidiana, EL Sistema_de_Recomendación DEBERÁ satisfacer la relación `distancia = raíz(sumatoria de c_j)` con una tolerancia absoluta de 0,000000001.
4. CUANDO la Métrica_de_Similitud sea Similitud_Coseno, EL Sistema_de_Recomendación DEBERÁ calcular para cada Característica_Numérica la Contribución_de_Característica `c_j = (x_j × y_j) / (Norma(x) × Norma(y))` sobre los Vectores_Ponderados comparados.
5. CUANDO la Métrica_de_Similitud sea Similitud_Coseno, EL Sistema_de_Recomendación DEBERÁ satisfacer la relación `similitud = sumatoria de c_j` con una tolerancia absoluta de 0,000000001.
6. CUANDO el Sistema_de_Recomendación produzca una Agrupación_de_Afinidad, EL Sistema_de_Recomendación DEBERÁ incluir para cada Cluster_de_Afinidad una cantidad positiva de Entidades cuya suma sea igual a la cantidad total de Entidades_Aptas.
7. CUANDO el Sistema_de_Recomendación produzca una Agrupación_de_Afinidad, EL Sistema_de_Recomendación DEBERÁ incluir para cada Cluster_de_Afinidad la media aritmética de cada Característica_Numérica en sus valores normalizados y en sus valores ponderados, verificable con una tolerancia absoluta de 0,000000001.
8. CUANDO el Sistema_de_Recomendación produzca cualquier resultado analítico, EL Sistema_de_Recomendación DEBERÁ asociar la normalización, los Parámetros_de_Transformación, los Pesos_de_Característica, la Métrica_de_Similitud y la Semilla_de_Ejecución de la Configuración_Efectiva.

### Requisito 7: Serialización, formato legible y exportación

**Historia de usuario:** Como Usuario, quiero exportar datos y resultados en formatos interoperables, para revisar, compartir o reutilizar el análisis.

#### Criterios de aceptación

1. CUANDO el Usuario solicite una exportación JSON, EL Serializador DEBERÁ generar JSON conforme a RFC 8259, codificado en UTF-8 sin marca de orden de bytes, con un Registro por Entidad o elemento de resultado y con todos los campos aplicables.
2. CUANDO el Usuario solicite una exportación CSV, EL Serializador DEBERÁ generar CSV conforme a RFC 4180, codificado en UTF-8 sin marca de orden de bytes, con terminadores de línea CRLF, una fila de encabezados, un Registro por Entidad o elemento de resultado y todos los campos aplicables.
3. CUANDO el Formateador_Legible reciba un Conjunto_de_Datos o un resultado analítico, EL Formateador_Legible DEBERÁ producir una representación determinista con los campos en el orden del Esquema_Declarado y las colecciones en orden de posición o, cuando no exista posición, por Identificador_de_Entidad ascendente según valores Unicode.
4. CUANDO el Sistema_de_Recomendación exporte una Recomendación_por_Entidad, EL Serializador DEBERÁ preservar el orden por posición y todos los campos de cada recomendación, incluidos la posición, el Identificador_de_Entidad y el valor de comparación.
5. CUANDO el Sistema_de_Recomendación exporte una Lista_Curada, EL Serializador DEBERÁ preservar el orden por posición y todos los campos de cada elemento, incluidos la posición y el Identificador_de_Entidad.
6. CUANDO el Sistema_de_Recomendación exporte una Agrupación_de_Afinidad, EL Serializador DEBERÁ ordenar los Registros por Identificador_de_Entidad ascendente según valores Unicode y preservar la asignación de Cluster_de_Afinidad de cada Entidad_Apta.
7. CUANDO un Conjunto_de_Datos válido se serialice a JSON y se vuelva a importar, EL Importador DEBERÁ producir un Conjunto_de_Datos equivalente en cantidad y orden de Entidades, Identificadores_de_Entidad, nombres y orden de campos, tipos y valores, manteniendo la distinción entre valor nulo, valor vacío y campo ausente.
8. CUANDO un Conjunto_de_Datos válido se serialice a CSV y se vuelva a importar con el mismo Esquema_Declarado, EL Importador DEBERÁ producir un Conjunto_de_Datos equivalente en cantidad y orden de Entidades, Identificadores_de_Entidad, nombres y orden de campos, tipos y valores, manteniendo según el Esquema_Declarado la distinción entre valor nulo, valor vacío y campo ausente.

### Requisito 8: Capacidad, rendimiento y protección de datos

**Historia de usuario:** Como Usuario, quiero analizar colecciones de tamaño práctico con resultados reproducibles y controlados, para usar la funcionalidad de manera confiable.

#### Criterios de aceptación

1. MIENTRAS un Conjunto_de_Datos contenga entre 2 y 100.000 Entidades y entre 2 y 100 Características_Numéricas, EL Sistema_de_Recomendación DEBERÁ completar la validación con una Memoria_Residente_Máxima menor o igual que 16 GB en el Entorno_de_Referencia sin carga concurrente.
2. CUANDO el Usuario solicite una Recomendación_por_Entidad sobre 100.000 Entidades y 100 Características_Numéricas ya normalizadas, EL Sistema_de_Recomendación DEBERÁ completar cada una de cinco ejecuciones consecutivas con un Tiempo_Extremo_a_Extremo menor o igual que 5 segundos en el Entorno_de_Referencia sin carga concurrente.
3. CUANDO el Usuario solicite una Agrupación_de_Afinidad de entre 2 y 100 Clusters_de_Afinidad sobre 100.000 Entidades y 100 Características_Numéricas, EL Sistema_de_Recomendación DEBERÁ completar cada una de cinco ejecuciones consecutivas con un Tiempo_Extremo_a_Extremo menor o igual que 120 segundos en el Entorno_de_Referencia sin carga concurrente.
4. CUANDO el Usuario ejecute un análisis, EL Sistema_de_Recomendación DEBERÁ procesar el Conjunto_de_Datos exclusivamente dentro del entorno de ejecución configurado por el Usuario, sin transmitir ni copiar datos a entornos externos.
5. CUANDO el Sistema_de_Recomendación reciba una solicitud de análisis, EL Sistema_de_Recomendación DEBERÁ validar los límites de cantidad de Entidades, Características_Numéricas y Clusters_de_Afinidad antes de iniciar el análisis.
6. SI el Conjunto_de_Datos o la Cantidad_de_Clusters supera los límites declarados, ENTONCES EL Sistema_de_Recomendación DEBERÁ rechazar el análisis, informar los límites y los valores recibidos, omitir resultados parciales y conservar el Conjunto_de_Datos cargado.
7. CUANDO el Usuario repita un análisis con el mismo Conjunto_de_Datos, la misma Configuración_Efectiva y la misma Versión_del_Sistema, EL Sistema_de_Recomendación DEBERÁ conservar los mismos Identificadores_de_Entidad, orden, valores del resultado y asignaciones de Cluster_de_Afinidad aplicables al tipo de resultado.
8. CUANDO el Sistema_de_Recomendación verifique los límites o mida la Memoria_Residente_Máxima y el Tiempo_Extremo_a_Extremo, EL Sistema_de_Recomendación DEBERÁ realizar la verificación y las mediciones únicamente con los recursos del Entorno_de_Referencia, sin requerir infraestructura externa.
