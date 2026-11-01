# Análisis de participación ciudadana — INE

Análisis exploratorio y segmentación de patrones de participación electoral con datos de los conteos censales de participación ciudadana del INE.

> Alcance: los perfiles describen secciones electorales y cohortes agregadas por estado; no identifican personas. La participación ajustada se calcula como SV / (LN - NS).

## Notebooks

| Notebook | Descripción |
|---|---|
| EDA.ipynb (./EDA.ipynb) | Consolida los CSV, registra validaciones de calidad y genera tablas y gráficas descriptivas. |
| clustering_ciudadania.ipynb (./clustering_ciudadania.ipynb) | Construye perfiles y aplica clustering por sección electoral y por cohorte de nacimiento y estado. |

## Requisitos

- Python 3.9 o posterior
- Jupyter Notebook o JupyterLab

pip install pandas numpy matplotlib scikit-learn pyarrow

`pyarrow` habilita la salida Parquet. Sin esa biblioteca, el EDA guarda el consolidado como CSV comprimido.

## Datos de entrada

Coloca los CSV de conteos censales en la carpeta que se indica en `ROOT`, al inicio de `EDA.ipynb`. El notebook busca recursivamente archivos cuyo nombre contenga `conteoscensales`.

Los CSV deben incluir estas columnas:

AELEC, FELECCION, EDOCVE, EDONOM, MPIOCVE, MPIONOM, SECCION,
TIPOSEC, DEF, DEL, SEXO, EDAD, LN, SV, NV, NS

El EDA intenta leer archivos codificados en UTF-8, UTF-8 con BOM o Latin-1. El log registra alertas por valores faltantes, duplicados, años o entidades inesperados e inconsistencias en LN = SV + NV + NS.

## Configuración y ejecución

Las rutas incluidas en los notebooks corresponden a otro equipo. Actualízalas antes de ejecutar:

1. En EDA.ipynb, configura ROOT con la carpeta de los CSV. La salida se guarda en salida_eda, junto a esa carpeta.
2. Ejecuta todas las celdas de EDA.ipynb.
3. En clustering_ciudadania.ipynb, configura OUT_DIR con la ruta a salida_eda.
4. Ejecuta las celdas del notebook de clustering en orden.

El clustering necesita el consolidado generado por el EDA. Puedes fijar el número de clusters mediante K_FINAL_SECCION y K_FINAL_COHORTE en la configuración de ese notebook.

## Resultados

El EDA genera en salida_eda/:

- conteos_consolidado.parquet o conteos_consolidado.csv.gz
- log_validaciones.txt
- tablas/: resúmenes de participación por año, estado, sexo, edad, tipo de sección, municipio y sección
- graficas/: visualizaciones de participación nacional, territorial y demográfica

El clustering guarda sus resultados en salida_eda/clustering/, organizados por unidad de análisis. Incluye variables construidas, métricas para seleccionar el número de clusters, perfiles, asignaciones y gráficas.

## Criterios y limitaciones

- Secciones: se excluyen observaciones sección-año con más de 50 % de votos no especificados (NS) o menos de 100 registros válidos. Cada sección debe contar con al menos cuatro años válidos, incluyendo una elección presidencial y una intermedia.
- Cohortes: se agrupan edades en generaciones de cinco años por estado. La participación observada se compara con la esperada para la misma edad y elección a nivel nacional. Se excluyen edades mayores de 90 años y cohortes que no cumplen los mínimos de observaciones.
- Las tasas se suavizan hacia la media. En el análisis por sección, las variables se recortan entre los percentiles 1 y 99, se estandarizan y se reducen mediante PCA.
- Una silueta baja puede indicar que la participación forma un gradiente continuo en vez de grupos claramente separados. Interpreta las métricas y los perfiles antes de nombrar los clusters.
- Los cambios o divisiones de secciones entre elecciones pueden afectar qué unidades cumplen los requisitos de inclusión.

