# Prueba técnica, análisis de mercado — Informe técnico

## Estructura del informe
- [Introducción](#introducción)
- [Metodología](#1-metodología)
- [Análisis Temporal y Pronóstico de Ventas](#2-análisis-temporal-y-pronóstico-de-ventas)
- [Segmentación de Clientes](#3-segmentacion-de-clientes)
- [Evaluación del Desempeño Logístico](#4-evaluación-del-desempeño-logístico)
- [Análisis Económico y Rentabilidad](#5-análisis-económico-y-rentabilidad)
- [Análisis de Productos](#6-análisis-de-productos)
- [Análisis geográfico](#7-análisis-geográfico)
- [Autor](#autor)

## Introducción

Este trabajo se desarrolla como una práctica que pretende evaluar la capacidad del aspirante de realizar un análisis de ventas para transformar registros históricos de órdenes de compra en insights estratégicos que faciliten la toma de decisiones.

El conjunto de datos utilizado cuenta con un extenso historial de órdenes de compra entre 2021 y 2024, distribuidas geográficamente y abarcando múltiples productos y métodos de envío. Se requiere entonces extraer valor de esta información y traducirla en recomendaciones concretas para la empresa.

En particular, se plantean los siguientes objetivos específicos:

- **Explorar el dataset** mediante análisis visual y estadístico, entendiendo su distribución de clases y calidad de los datos, para su preprocesamiento.
- **Análisis Temporal y Pronóstico de Ventas** Desarrollar modelos predictivos SARIMA que capturen tendencias y estacionalidad, permitiendo planificación financiera, gestión de inventario y asignación de recursos con anticipación.
- **Segmentación de Clientes** Identificar grupos homogéneos de clientes mediante clustering (K-Means) y análisis RFM (Recency, Frequency, Monetary), permitiendo estrategias de marketing diferenciadas.
- **Evaluación del Desempeño Logístico** Analizar tiempos de entrega, costos de envío y rentabilidad por método logístico para identificar cuellos de botella y oportunidades de optimización operativa.
- **Análisis Económico y Rentabilidad, producto y geográfico** Evaluar márgenes de beneficio por producto, categoría y región geográfica; cuantificar el impacto de descuentos; e identificar productos estrella vs. productos problemáticos mediante matriz BCG.

## 1. Metodología

### 2.1. Dataset

Se utilizó el dataset data.xlsx en formato excel con la siguiente distribución de hojas:
- **Ordenes**: 51290 filas x 14 columnas
- **Región**: 152 filas x 3 columnas
- **Productos**: 10768 filas x 4 columnas

El dataset presenta 28 valores nulos en Nombre_del_cliente y 1 Ventas, Cantidad, Descuento, Ganancia y Costo_de_envío. Además, presenta un valor atípico en País para  'Colombia-'. 
2 Valores atípicos para las columnas de Fecha_de_la_orden 2034.

### 2.2. Preprocesamiento

1. **Valores nulos**: 

Cálculo de asimetría de las columnas:

Asimetría de Ventas: 8.1380
Asimetría de Cantidad: 1.3604
Asimetría de Descuento: 1.3877
Asimetría de Ganancia: 4.1581
Asimetría de Costo_de_envío: 5.8632

Se sustituirán los valores nulos con la mediana del conjunto de datos, dado que los datos presentan gran dispersión, mostrando una diferencia significativa entre los mínimos y máximos, lo que se refleja en una distribución poco uniforme, con valores de asimetría diferentes de 1.

Para los clientes, al ser un dato categórico, se sustituirá por un nombre genérico: Cliente_(# index).

2. **Valores atípicos**: 
![Distribución año](assets/figures/01_exploratory/01_distribucion_fecha.png)

La distribución de los datos se encuentran entre el año 2021 y 2024. Existen dos valores atípicos para el 2034. Por su baja frecuencia y ausencia de continuidad temporal, se excluirán del análisis para no distorsionar tendencias y modelos.

3. **Métricas adicionales**: 
Se añaden métricas de Margen_beneficio, Precio_unitario y Ventas_netas.
![Matriz Correlación](assets/figures/01_exploratory/02_matriz_correlacion.png)
Se realiza la matriz para identificar la correlación de las nuevas métricas añadidas. Revelando que Margen_beneficio tiene relación inversa fuerte con los descuentos, el precio unitario relación directa con el precio y precio_unitario, y las ventas_netas tiene una relación directa con ganancia y costo_de_envio.

## 2. Análisis Temporal y Pronóstico de Ventas

Se realiza una descomposición temporal para determinar patrones de temporalidad en los datos:

![Descomposición temporal](assets/figures/02_features/01_descomposicion_temporal.png)

Observamos una tendencia creciente, con un patrón de picos en meses como junio, septiembre y crecimiento constantes de noviembre a diciembre, con caídas en enero y prolongación para febrero, disminución en abril y otra caída fuerte en octubre.

### 2.1. Modelo pronóstico de ventas

1. **Modelo SARIMA**: Se aplicó el modelo SARIMA para la predicción de los siguientes meses en la temporalidad de órdenes de compra. Las variables a tener en cuenta fueron las Ventas, Ganancia y ID_orden (cuenta de órdenes de compra).

2. **Búsqueda por grilla**: Se aplicó búsqueda por grilla para encontrar los parámetros óptimos para el modelo SARIMA.


### 2.2. Entrenamiento y Evaluación

**Búsqueda en grilla**
- 324 combinaciones probadas. Se excluyeron resultados en AIC y BIC extremos (< -100, > 1000) y valores con un MAPE inferior a 50%.
## Comparación de Modelos SARIMA

| Variable   | Modelo                         | AIC    | BIC    | MAPE     |
|-----------|---------------------------------|--------|--------|----------|
| Ventas    | (0,1,0) × (2,1,1,12)            | 31.19  | 23.19  | 9.78%    |
| Ganancia  | (2,0,0) × (0,1,2,12)            | 30.28  | 20.28  | 21.67%   |
| ID_orden  | (0,1,0) × (2,1,0,12)            | -8.76  | -14.76 | 7.69%    |

Mejores resultados en equilibrio entre AIC, BIC y MAPE (menor valor en el error porcentual medio de la predicción)

**Modelos SARIMA**
![SARIMA ventas](assets/figures/02_features/02_sarima_ventas.png)
![SARIMA ganancias](assets/figures/02_features/03_sarima_ganancias.png)
![SARIMA ordenes](assets/figures/02_features/04_sarima_ordenes.png)

Se captó el patrón asociado a la temporalidad, con un respectivo incremento debido a la tendencia creciente de los datos.

## 3. Segmentación de Clientes

### 3.1. Modelo para segmentación

**Normalización de clases**
- El dataset fue normalizado a través de StandardScaler para la variables que utilizará el modelo: 'Ventas_totales', 'Cantidad_total', 'Ganancia_total', 'Num_ordenes'

**K óptimo de clusters**
Se hace uso del método del codo para encontrar k óptimo:
![método codo](assets/figures/02_features/05_codo.png)
Se determina hacer uso de 4 clusters.

**Métodos de clustering**
Se realizan pruebas para diferentes métodos de clustering haciendo uso de los modelos Kmeans, GMM, Agglomerative, Spectral, DBSCAN, HDBSCAN y KMedoids.
![Métodos clustering](assets/figures/02_features/06_metodos_clustering.png)

Dado el resultado del coeficiente de silueta y la visual de distribución de los grupos, el mejor método para este set de datos es el Kmeans.


### 3.2. Resultados de K-Means

Segmentación por KMeans – Estadísticas por Cluster

| Segmento_KMeans | Num_clientes | Ventas_prom | Cantidad_prom | Ganancia_prom | Ordenes_prom | Descuento_prom | Recency_prom | Frequency_prom | Monetary_prom | RFM_prom |
|-----------------|--------------|-------------|----------------|----------------|----------------|-----------------|---------------|------------------|----------------|-----------|
| 0               | 326          | 17053.46    | 250.66         | 1678.69        | 71.72          | 0.15            | 20.53         | 71.72            | 17053.46       | 10.69     |
| 1               | 348          | 12107.69    | 182.12         | 1238.13        | 53.47          | 0.15            | 25.81         | 53.47            | 12107.69       | 6.76      |
| 2               | 119          | 23974.57    | 276.89         | 4131.90        | 77.41          | 0.12            | 20.76         | 77.41            | 23974.57       | 12.41     |
| 3               | 30           | 534.76      | 8.60           | -21.03         | 2.93           | 0.19            | 1170.17       | 2.93             | 534.76         | 3.10      |

Los segmentos están clasificados en la siguiente escala:

  - Clientes premium: 119 clientes, $23,974 promedio - Con alta frecuencia de compra, volumen sostenido y excelente rentabilidad. Representan el núcleo estratégico del negocio.

  - Cliente alto valor: 326 clientes, $17,053.46 promedio - Clientes valiosos y constantes, con comportamiento sólido. No tienen los niveles del segmento Premium, pero aportan ingresos estables y rentables.

  - Medio: 348 clientes, $12,107.69 promedio - Clientes de valor medio, con compras menos frecuentes y menor volumen. Son importantes porque podrían migrar a segmentos superiores con estrategias de fidelización.

  - Bajo Valor: 30 clientes, $534.76 promedio - Clientes con muy baja frecuencia y rentabilidad negativa. Pueden ser clientes esporádicos o que generan pérdidas por descuentos altos o costos asociados.


## 4. Evaluación del Desempeño Logístico

Se evalúan las métricas:
![Desempeño logístico](assets/figures/02_features/07_logistico.png)

Se encontraron los siguientes KPIs de desempeño logístico
Tiempo promedio de entrega: 3.97 días
Método más usado: Standard Class
Costo promedio de envío: $26.38

Hubo entregas retrasadas en Same Day. este mismo método de envío es el más costoso. Adicionalmente, el Estándar Class es el único método que generó ganancias por encima del costo promedio para cada orden.


### 5. Análisis Económico y Rentabilidad
![Desempeño económico](assets/figures/02_features/08_economico.png)

Se encontró que las órdenes con pérdidas son de 12542 (24.5%), representando una pérdida total acumulada de $-920,181.85. Posiblemente se deba al descuento promedio en órdenes con pérdida que alcanzan los 45.1%. 

Existen muchas ventas sin ganancias, con descuentos altos.


## 6. Análisis de Productos

Se realizó una distribución por categoría de precio tomando en cuenta el modelo BGC (Boston consulting group), agrupando los productos en cuatro categorías (Estrellas, Vacas Lecheras, Interrogación, Perros). Estos dependerán de sus ventas_totales y margen_promedio en comparación con la mediana:

- >= mediana_margen, Estrella.
- >= mediana_ventas y < mediana_margen, Vacas Lecheras.
- < mediana_ventas y < mediana_margen, Perro.
- Cualquier otro caso, interrogante.

Aproximadamente el 90% de los productos se encuentran entre Estrella y vacas lecheras, lo que indica que la mayoría de productos generan margen para la empresa.

![Desempeño productos](assets/figures/02_features/09_producto.png)
El top de productos por ganancia y ventas están encabezados por el componente tecnológico, con la impresora Canon imageClass 2200. 
Por su parte, la matriz de productos demuestra que existen todavía muchos productos con margen de beneficio negativos. Esto responde a que el margen global de ganancias no supere el 11%. Se debe establecer estrategias alrededor de los costos y precios de estos productos.

## 7. Análisis geográfico
![Desempeño paises](assets/figures/02_features/10_geografico.png)
Se evidencia ventas totales y número de órdenes superiores en los departamentos de England y California. El margen de beneficio está encabezado por Tottori. Además, los descuentos mayores son en promedio del 70% en distintos departamentos. 

## Autor:

* Jorge Alexander Orrego Puerta.

