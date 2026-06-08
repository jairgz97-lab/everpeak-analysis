# everpeak-analysis
# EverPeak Retail - Análisis Exploratorio, Auditoría de Datos y Pipeline de Limpieza

Este repositorio contiene el desarrollo de un **Análisis Exploratorio de Datos (EDA)** y el diseño de un **Pipeline de Limpieza Automatizado** aplicado al conjunto de datos de ventas de **EverPeak Retail**. El objetivo principal de este proyecto es diagnosticar la calidad de la información, corregir anomalías estructurales y preparar el dataset para futuros análisis estadísticos avanzados o modelos predictivos, asegurando el máximo rigor metodológico.

---

## 📋 Resumen Ejecutivo del Proyecto

El análisis del dataset original de EverPeak Retail reveló que, detrás de una apariencia inicial de completitud, existían fallas críticas de captura y lógicas que habrían sesgado cualquier toma de decisiones si se hubieran ignorado. A través de este proyecto, se implementó una arquitectura defensiva en Python (Pandas/NumPy) que audita, limpia y documenta sistemáticamente estas inconsistencias sin destruir la realidad financiera del negocio.

### 🔍 Hallazgos Críticos Identificados

1. **Fallo Estructural en el Registro de Volúmenes (`quantity == 0`):** Se identificó que el **59.22% del dataset (2,966 filas)** presentaba un valor de `0` en la columna de cantidad, a pesar de registrar transacciones financieras activas y positivas en el valor total de la orden (`order_value`). 
2. **Inconsistencia de Integridad Lógica ($Precio \times Cantidad \neq Valor Total$):** Al auditar los registros con cantidades mayores a cero (como la fila 64), se detectó que el monto de `order_value` no equivale a la multiplicación directa de `price` y `quantity`, existiendo un desfase sistemático de aproximadamente un ~6%. Se infiere la presencia de costos operativos no desglosados (como impuestos locales o tarifas de envío).
3. **Valores Centinela Ocultos (`?`):** La columna `product_category` contenía el carácter `?` actuando como un valor nulo encubierto, el cual evadía los filtros tradicionales de detección de nulos (`.isnull()`) debido a la presencia de espacios en blanco invisibles (`" ? "`).
4. **Registros Basura en Datos Demográficos (`customer_age`):** Se detectaron valores imposibles para la edad de los clientes (`-999`, `999`, `0`, `1`), que distorsionaban severamente las métricas descriptivas de la población.

---

## 🛠️ Arquitectura del Pipeline de Limpieza

Toda la lógica de saneamiento se encapsuló en una función modular y reproducible llamada `pipeline_limpieza_everpeak`. Esta arquitectura sigue las mejores prácticas de ingeniería de datos:

* **Estrategia para Fechas Faltantes:** Se eliminaron de forma selectiva las filas con fechas nulas mediante `.dropna(subset=['order_date'])`. Esto resolvió un efecto de *errores concurrentes*, eliminando colateralmente filas que carecían tanto de fecha como de volumen de compra.
* **Tratamiento MCAR (Missing Completely at Random):** Los valores basura en `customer_age` se aislaron e imputaron utilizando la **mediana de la población**, una métrica robusta frente a sesgos extremos.
* **Saneamiento de Texto Extremo:** Se aplicó un proceso de normalización de cadenas de texto mediante `.str.strip()` para eliminar espacios vacíos y se reemplazaron de manera segura las subcadenas `?` por la etiqueta `"unknown"`.
* **Preservación Lingüística:** Se optó deliberadamente por mantener el carácter `ñ` en la columna `año` (en lugar de forzar una conversión a "ano"), priorizando la precisión lingüística y el contexto regional del reporte.

---

## 📊 Contraste Metodológico: Detección de Outliers

Para la variable crítica de negocio `order_value`, se implementó y documentó un análisis comparativo entre dos técnicas de detección de valores atípicos:

| Módulo Estadístico | Umbral Aplicado | Outliers Detectados | Justificación de Comportamiento |
| :--- | :--- | :---: | :--- |
| **Z-Score Clásico** | $3\sigma$ (3 Desviaciones Estándar) | **49** | La media y la desviación estándar se ven severamente arrastradas hacia la derecha por la cola larga de ingresos, ensanchando el umbral y permitiendo la fuga de anomalías. |
| **Rango Intercuartílico (IQR)** | $Q3 + 1.5 \times IQR$ | **114** | Al basarse en la mediana y los cuartiles (métricas de posición), es inmune a los valores extremos, manteniendo una barra firme de control sobre la distribución real del comprador común. |

> 💡 **Decisión Analítica:** Para efectos de modelado de la distribución del cliente estándar, se seleccionó el enfoque **IQR** por su robustez ante distribuciones comerciales con asimetría positiva. Los 114 outliers representan compras institucionales o de mayoreo que deben ser analizadas como un segmento de negocio independiente.

---

## 📈 Decisiones de Negocio y Próximos Pasos

A diferencia de un enfoque automatizado e ingenuo que altera o borra datos para "forzar" que las cuentas cuadren, este proyecto defiende una postura de **auditoría transparente**:
* **Conservación Financiera:** Las columnas `price` y `order_value` se mantuvieron intactas para no adulterar el flujo de caja histórico reportado por la empresa. Las inconsistencias matemáticas se asentaron formalmente en las notas de auditoría.
* **Próxima Fase:** Distribución visual y segmentación. El dataset limpio y estructurado está listo para la fase de visualización, donde se generarán histogramas recortados al límite del IQR (\$431.25) y análisis del alto rendimiento financiero detectado en el segmento de ubicación `"unknown"`.

---

## 🚀 Tecnologías Utilizadas

* **Python 3.x**
* **Pandas** (Manejo y transformación de estructuras de datos)
* **NumPy** (Operaciones vectoriales y manejo de arreglos numéricos)
* **Jupyter Notebook** (Entorno de desarrollo e informes interactivos)

---
*Este proyecto forma parte de mi portafolio profesional de Análisis de Datos, desarrollado con el rigor y estándares técnicos del bootcamp de TripleTen.*
