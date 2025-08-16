![Blue Bikes Logo](assets/bluebikes_logo.png)

## Descripción
**Proyecto Final – Blue Bikes (Data Mining – MIAV1E2)**  
Repositorio del trabajo final para la asignatura **Data Mining – MIAV1E2**.

Incluye:
- **Notebook** con el código limpio y documentado.
- **Informe** con resumen de la base de datos, procesos de limpieza y transformación, metodología de modelado, resultados, conclusiones y recomendaciones.
- **Presentación en PDF** con la síntesis del proyecto.
- **Diccionario de datos en PDF**.
- **Base de datos** (o enlace a la fuente).
- Archivos auxiliares y estructura para reproducibilidad.

## Autoría
- Estudiantes:
  - *Nicolas Oporto*
  - *Joseph Thenier*
  - *Carolina Bello*
  - *Luis Martinez*
  - *Oscar Loayza*
- Carrera/Asignatura: **Data Mining – MIAV1E2**  
- Docente: *MSc. Renzo Franck Claure Aracena*


### Descripción corta (para el campo "About" de GitHub)
Proyecto final 'Blue Bikes' – Data Mining (MIAV1E2): notebook, informe, presentación y diccionario de datos.


Repositorio del **Proyecto Final de Data Mining – MIAV1E2** (docente: *MSc. Renzo Franck Claure Aracena*).

## Entregables
- **Notebook** con el código limpio y documentado (`notebooks/Proyecto_final_Blue_Bikes.ipynb`).
- **Informe** que resuma:
  - Información de la base de datos
  - Problemas resueltos (limpieza y transformación)
  - Métodos y modelos utilizados
  - Resultados obtenidos
  - Conclusiones y recomendaciones
- **Presentación (PDF)**: `reports/Presentacion_Blue_Bikes.pdf`
- **Diccionario de datos (PDF)**: `reports/Diccionario_de_Datos_Blue_Bikes.pdf`
- **Base de datos** utilizada (o el **link** a la fuente). Coloque los archivos en `data/` o añada el enlace en el informe.
## Estructura del repositorio
```
miav1e2-blue-bikes/
├─ notebooks/
│  └─ Proyecto_final_Blue_Bikes.ipynb
├─ reports/
│  └─ Informe_Proyecto_Blue_Bikes.md
├─ data/
│  └─ .gitkeep
├─ src/
├─ requirements.txt
└─ .gitignore
```

## Configuración rápida
1. **Crear entorno** (opcional):
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # Windows: .venv\Scripts\activate
   ```

2. **Instalar dependencias**:
   ```bash
   pip install -r requirements.txt
   ```

3. **Abrir el notebook**:
   ```bash
   jupyter lab notebooks/Proyecto_final_Blue_Bikes.ipynb
   ```

## Uso con PySpark (Java requerido)
Si vas a ejecutar el notebook con **PySpark**, es necesario tener **Java (JDK)** instalado y configurar algunas opciones del runtime.

### 1) Instalar Java
- Recomendado: **JDK 11 o 17**.  
- Verifique instalación:
  ```bash
  java -version
  ```
- (Opcional) Defina `JAVA_HOME` y agregue `JAVA_HOME/bin` al `PATH`.

### 2) Instalar PySpark
```bash
pip install pyspark
```

### 3) Configuración recomendada de SparkSession
Usamos opciones `--add-opens` (Java 9+) para compatibilidad con el sistema de módulos de Java cuando se ejecuta Spark localmente.

```python
from pyspark.sql import SparkSession

java_options = "--add-opens java.base/java.lang=ALL-UNNAMED --add-opens java.base/java.lang.invoke=ALL-UNNAMED --add-opens java.base/java.lang.reflect=ALL-UNNAMED --add-opens java.base/java.io=ALL-UNNAMED --add-opens java.base/java.net=ALL-UNNAMED --add-opens java.base/java.nio=ALL-UNNAMED --add-opens java.base/java.util=ALL-UNNAMED --add-opens java.base/java.util.concurrent=ALL-UNNAMED --add-opens java.base/java.util.concurrent.atomic=ALL-UNNAMED --add-opens java.base/sun.nio.ch=ALL-UNNAMED --add-opens java.base/sun.nio.cs=ALL-UNNAMED --add-opens java.base/sun.security.action=ALL-UNNAMED --add-opens java.base/sun.util.calendar=ALL-UNNAMED"

spark = SparkSession.builder \    .appName("Unificar Esquemas Hubway") \    .master("local[2]") \    .config("spark.driver.host", "127.0.0.1") \    .config("spark.driver.bindAddress", "127.0.0.1") \    .config("spark.sql.execution.arrow.pyspark.enabled", "false") \    .config("spark.python.worker.timeout", "600") \    .config("spark.network.timeout", "600s") \    .config("spark.python.worker.reuse", "false") \    .config("spark.driver.memory", "2g") \    .config("spark.sql.shuffle.partitions", "4") \    .config("spark.driver.extraJavaOptions", java_options) \    .config("spark.executor.extraJavaOptions", java_options) \    .getOrCreate()

spark.sparkContext.setLogLevel("WARN")
```

**Notas:**
- `local[2]` limita el uso de recursos; ajuste según su equipo.
- Si utiliza **Java 8**, normalmente no se requieren las banderas `--add-opens`. Para **Java 11/17**, manténgalas.
- Deshabilitamos **Arrow** en PySpark para evitar incompatibilidades locales.


## Datos
- Coloque la(s) base(s) en `data/` (no se versionan por defecto) o incluya el **enlace** a la fuente en el informe.
- Si los datos son grandes/sensibles, suba un **muestra** y un **script**/enlace para reproducir la descarga.

## Reproducibilidad
- Fije semillas aleatorias en el notebook y documente versiones clave en el informe.
- Exporte gráficos/tablas relevantes a `reports/` cuando corresponda.

