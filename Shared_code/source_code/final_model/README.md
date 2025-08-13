# 🚲 Predicción de Tipos de Usuarios de Bicicletas Compartidas



Este proyecto se enfoca en mejorar la clasificación de usuarios de bicicletas compartidas entre "miembros" y "casuales", enfrentando el desafío de un conjunto de datos altamente desbalanceado.



## 🔍 Resumen de la Iteración Final



### 1. Diagnóstico del Problema  

Se detectó que la alta precisión del modelo original era engañosa debido a un **grave desequilibrio de clases**, lo que provocaba que el modelo ignorara la clase minoritaria ("casual").



### 2. Aplicación de Solución  

Se implementó la **ponderación de clases**, lo que obligó al modelo a prestar más atención a la clase minoritaria. Inicialmente se usó **Logistic Regression (LR)**, pero mostró signos de **sobrecorrección**.



### 3. Actualización del Algoritmo  

Se reemplazó LR por un **GBTClassifier (Gradient-Boosted Trees)**, un modelo más robusto y capaz de manejar mejor la complejidad inducida por los pesos de clase.



### 4. Ingeniería de Características  

Se agregaron nuevas variables predictivas, como `avg_speed_kmh`, que proporcionaron señales más sólidas y relevantes para el modelo.



### 5. Ajuste y Optimización  

Se utilizó **CrossValidator** para realizar búsqueda de hiperparámetros y encontrar la configuración óptima del modelo, maximizando su desempeño final.



---



## 📈 Resultados



- **F1 Score Final**: `0.7787` — la más alta obtenida con un modelo equilibrado hasta el momento.

- **Mejor desempeño general**:  

&nbsp; El modelo ajustado identificó **más usuarios "miembros" y "casuales" correctamente** que versiones anteriores. La matriz de confusión mostró **menos errores en todas las categorías**, lo cual indica un modelo claramente superior.



---



## 🚀 Próximos Pasos



- **Ampliar la búsqueda de hiperparámetros**: Explorar una gama más amplia de configuraciones para el `GBTClassifier` mediante `CrossValidator`.

- **Ingeniería avanzada de características**: Incluir variables externas como condiciones meteorológicas o feriados, que probablemente tengan un impacto importante en el comportamiento de los usuarios.



---






