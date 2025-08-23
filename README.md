![Blue Bikes Logo](assets/bluebikes_logo.png)

# Bluebikes Boston – Proyecto Final (Data Mining – MIAV1E2)

**Repositorio oficial del trabajo final** de la asignatura **Data Mining – MIAV1E2**.  
Incluye notebooks, informes, diccionario de datos, artefactos de modelado y una estructura reproducible del proyecto.

> **Resumen**: Integramos y limpiamos un histórico multimillonario de viajes de **Bluebikes Boston**, unificando 3 esquemas de datos (2015–2025), construimos una **base minable** en formato parquet, realizamos **EDA**, diseñamos **features** temporales/geoespaciales y entrenamos un modelo de **pronóstico horario por estación** siguiendo **CRISP‑DM**.

---

## Tabla de contenidos
- [Estructura del repositorio](#estructura-del-repositorio)
- [Instalación y puesta en marcha](#instalación-y-puesta-en-marcha)
  - [Requisitos](#requisitos)
  - [Entorno](#entorno)
  - [Java & PySpark](#java--pyspark)
- [Datos](#datos)
- [Metodología (CRISP‑DM) y pipeline](#metodología-crispdm-y-pipeline)
- [Cómo reproducir los resultados](#cómo-reproducir-los-resultados)
- [Resultados y entregables](#resultados-y-entregables)
- [Diccionario de datos](#diccionario-de-datos)
- [Reproducibilidad y calidad](#reproducibilidad-y-calidad)
- [Autoría y créditos](#autoría-y-créditos)
- [Licencia](#licencia)

---

## Estructura del repositorio

```
miav1e2-blue-bikes/
├─ assets/
│  └─ bluebikes_logo.png
├─ data/
│  ├─ raw/         # fuentes originales (no versionadas por defecto)
│  ├─ interim/     # staging / integraciones temporales
│  ├─ curated/     # tabla de viajes limpia y unificada (parquet)
│  ├─ features/    # viajes con features derivadas
│  └─ aggregates/  # agregados por estación × hora
├─ notebooks/
│  ├─ 01_Proyecto_final_Blue_Bikes_Limpieza-EDA.ipynb
│  └─ 02_Proyecto_final_Blue_Bikes_Ing_Caracteristicas.ipynb
├─ docs/
│  ├─ DATA_DICTIONARY.md
│  
├─ reports/
│  └─ Informe_Proyecto_Blue_Bikes.pdf
├─ src/            # (opcional) utilidades/ETL/modelado
├─ requirements.txt
├─ README.md
└─ LICENSE
```

> Si tus archivos tienen otros nombres, mantén la estructura de carpetas y ajusta las rutas internas en los notebooks.

---

## Instalación y puesta en marcha

### Requisitos
- **Python 3.10+**
- **Java JDK 11/17** (para PySpark)
- **pip** (o **conda**)

### Entorno
```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
source .venv/bin/activate
pip install --upgrade pip
pip install -r requirements.txt
```

### Java & PySpark
Verifica Java:
```bash
java -version
```
Instala PySpark (si no viene en `requirements.txt`):
```bash
pip install pyspark
```
SparkSession sugerido (Java 11/17):
```python
from pyspark.sql import SparkSession

java_options = "--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.lang.invoke=ALL-UNNAMED --add-opens java.base/java.lang.reflect=ALL-UNNAMED --add-opens java.base/java.io=ALL-UNNAMED --add-opens java.base/java.net=ALL-UNNAMED --add-opens java.base/java.nio=ALL-UNNAMED --add-opens java.base/java.util=ALL-UNNAMED --add-opens java.base/java.util.concurrent=ALL-UNNAMED --add-opens java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.base/sun.nio.cs=ALL-UNNAMED --add-opens java.base/sun.security.action=ALL-UNNAMED --add-opens java.base/sun.util.calendar=ALL-UNNAMED"

spark = SparkSession.builder \\
    .appName("Bluebikes CRISP-DM") \\
    .master("local[2]") \\
    .config("spark.driver.host", "127.0.0.1") \\
    .config("spark.driver.bindAddress", "127.0.0.1") \\
    .config("spark.sql.execution.arrow.pyspark.enabled", "false") \\
    .config("spark.python.worker.timeout", "600") \\
    .config("spark.network.timeout", "600s") \\
    .config("spark.python.worker.reuse", "false") \\
    .config("spark.driver.memory", "2g") \\
    .config("spark.sql.shuffle.partitions", "4") \\
    .config("spark.driver.extraJavaOptions", java_options) \\
    .config("spark.executor.extraJavaOptions", java_options) \\
    .getOrCreate()

spark.sparkContext.setLogLevel("WARN")
```

---

## Datos

- Coloca los archivos **fuente** en `data/raw/`. Por defecto, estos **no se versionan** (ver `.gitignore`).  
- Si el volumen es grande, sube **muestras** representativas y deja scripts/notebooks de descarga.  
- La tabla unificada y limpia se persiste en **parquet** en `data/curated/` para análisis eficiente.

> Nota: En los notebooks se integran **tres esquemas** históricos, se filtran viajes **<60s** y **>24h**, se validan coordenadas (excluyendo 0/0) y se aplica un filtro geográfico por BBOX+buffer del área operativa.

---

## Metodología (CRISP‑DM) y pipeline

1. **Comprensión del negocio**: objetivos operativos (re‑balanceo, staffing) y analíticos.  
2. **Comprensión de los datos**: perfilado de esquemas/volumen y calidad inicial.  
3. **Preparación**: integración multi‑esquema, reglas de limpieza, persistencia en parquet.  
4. **Modelado**: agregados **estación × hora**, lags/ventanas/Fourier y **XGBoost** para predicción de `rides`.  
5. **Evaluación**: comparación vs. baseline ingenuo (lag semanal) y métricas (MAE, RMSE, wMAPE, NRMSE).  
6. **Despliegue/Próximos pasos**: automatización de ingesta, monitoreo de calidad y mejoras con clima/eventos.

---

## Cómo reproducir los resultados

1. **Abrir notebooks**:
   ```bash
   jupyter lab notebooks/01_Proyecto_final_Blue_Bikes_Limpieza-EDA.ipynb
   jupyter lab notebooks/02_Proyecto_final_Blue_Bikes_Ing_Caracteristicas.ipynb
   ```
2. **Notebook 01 (Limpieza + EDA)**:  
   - Detecta y unifica esquemas.  
   - Aplica reglas de limpieza y validación geográfica.  
   - Exporta `data/curated/trips.parquet` y gráficos/tablas de EDA (opcionalmente a `reports/`).

3. **Notebook 02 (Features + Agregados + Modelado)**:  
   - Enriquecimiento temporal/geoespacial.  
   - Construcción de `data/aggregates/hourly_station.parquet`.  
   - Entrenamiento y evaluación del modelo de demanda por hora/estación.

---

## Resultados y entregables

- **Informe técnico (DOCX)**: `reports/Informe_Bluebikes_CRISPDM.docx`  
- **Presentación (PDF)**: `reports/Presentacion_Blue_Bikes.pdf` *(opcional; agregar cuando esté lista)*  
- **Artefactos de datos**:
  - `data/curated/` – tabla limpia y minable (parquet)  
  - `data/features/` – viajes con features derivadas  
  - `data/aggregates/` – agregados estación × hora

---

## Diccionario de datos

- **Markdown**: `docs/DATA_DICTIONARY.md`  
- **CSV (machine‑readable)**: `docs/data_dictionary.csv`

Incluye tipos, dominios, reglas de calidad, derivaciones y ejemplos para **curated**, **features** y **aggregates**.

---

## Reproducibilidad y calidad

- **Versionado de dependencias** en `requirements.txt`.  
- **Semillas** controladas en notebooks y particionado por `periodo` (`YYYY-MM`).  
- **Checks sugeridos** (dbt‑style) para dominios y rangos (ver `docs/DATA_DICTIONARY.md`).  
- **Buenas prácticas**: particionar parquet por año/mes; registrar métricas de calidad (% nulos, outliers, viajes filtrados).

---

## Autoría y créditos

**Estudiantes:** Nicolas Oporto · Joseph Thenier · Carolina Bello · Luis Martínez · Oscar Loayza  
**Carrera/Asignatura:** Data Mining – MIAV1E2  
**Docente:** MSc. Renzo Franck Claure Aracena

**Créditos de datos:** Sistema de bicicletas compartidas **Bluebikes Boston**.  
Este repositorio es académico; ver la licencia de los datos originales antes de redistribuir.

---

## Licencia

- **Código**: MIT (sugerida).  
- **Datos**: sujetos a los términos de la fuente original (no se redistribuyen por defecto).

Si adoptas otra licencia, actualiza este apartado y añade `LICENSE` en la raíz.

