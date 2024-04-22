# Polars: Un Vistazo a la Eficiencia en el Procesamiento de Datos Tabulares
¿Qué es Polars?
Polars es una biblioteca de procesamiento de datos tabulares en Rust, diseñada para ofrecer un rendimiento excepcional y una API fácil de usar. Aunque es relativamente nueva en comparación con bibliotecas más establecidas como Pandas en Python, Polars ha ganado rápidamente popularidad debido a su velocidad y eficiencia en el procesamiento de grandes volúmenes de datos.

1. **Rendimiento Superior:**
Una de las principales ventajas de Polars es su rendimiento superior en comparación con Pandas. Esto se debe en gran medida a que Polars está escrito en Rust, un lenguaje de programación conocido por su velocidad y seguridad. La implementación eficiente de Polars permite realizar operaciones de procesamiento de datos significativamente más rápido que Pandas, lo que lo hace ideal para aplicaciones que requieren un procesamiento rápido de grandes conjuntos de datos.

2. **Funcionalidades Avanzadas:**
Polars ofrece una amplia gama de funcionalidades avanzadas para el procesamiento de datos tabulares. Esto incluye operaciones de filtrado, agrupación, unión y transformación de datos, así como la capacidad de manejar datos faltantes de manera eficiente. Además, Polars es compatible con operaciones de paralelismo y distribución, lo que permite aprovechar al máximo los recursos de hardware disponibles para acelerar aún más el procesamiento de datos.

3. **Integración con Python:**
Aunque Polars está escrito en Rust, ofrece una interfaz de Python que permite a los usuarios aprovechar todas sus funcionalidades directamente desde el entorno de Python. Esto facilita la integración de Polars en flujos de trabajo existentes de análisis de datos en Python, lo que permite a los usuarios aprovechar su rendimiento superior sin tener que aprender un nuevo lenguaje de programación.


En resumen, Polars es una biblioteca de procesamiento de datos tabulares que destaca por su rendimiento superior y sus funcionalidades avanzadas. Ya sea que estés trabajando con conjuntos de datos pequeños o grandes, Polars ofrece una solución eficiente y escalable para el procesamiento de datos en entornos de Python. Si buscas mejorar el rendimiento de tus aplicaciones de análisis de datos, Polars es definitivamente una herramienta que vale la pena explorar.

## Data
* En este escenario, estamos trabajando con un conjunto de datos extraído de Kaggle, un conocido repositorio de conjuntos de datos. El objetivo es explorar y preparar estos datos para su posterior almacenamiento en formato Parquet. Este formato es conocido por su eficiencia en el procesamiento de grandes volúmenes de datos, especialmente cuando se trabaja con sistemas de Big Data basados en Apache, como Apache Spark.<br>
Al usar Parquet, estamos aprovechando las ventajas que ofrece Apache en el procesamiento de Big Data. Parquet es un formato de archivo columnar, lo que significa que almacena los datos en columnas en lugar de filas. Esta estructura es altamente eficiente para consultas analíticas y operaciones de procesamiento en sistemas distribuidos. Además, Parquet ofrece compresión y codificación de datos avanzadas, lo que reduce el tamaño de los archivos y mejora aún más el rendimiento en entornos distribuidos.<br>
Al explorar y preparar los datos, estamos llevando a cabo una serie de pasos técnicos, como la limpieza de datos, la transformación de datos, el filtrado y la agregación. Estos pasos son fundamentales para garantizar la calidad y la coherencia de los datos antes de su procesamiento posterior.

**Instalar Polars**
```python
!pip install polars
```
<br>
Si estas trabajando en Google Colab te quiero informar que polars ya viene instalado en el entorno de ejecución
<br>

Recuerda antes de leer los datos, buscar la ruta de acceso donde tienes guardado el archivo csv
```python
df=pl.read_csv('/content/ventas-por-factura.csv')#Leer la path
df.head()
```
**Tranformaciones necesarias** <br>
En mi caso realice algunas transformaciones de tipo de datos necesarias para tener un buen esquema en los datos
```python
df = df.with_columns(pl.col("Monto").cast(pl.Utf8)) # Convertir la columna 'Monto' a tipo de datos Utf8 para que sea manipulable
df = df.with_columns(pl.col("Monto").str.replace(",", ".")) # Reemplazar las comas en la columna 'Monto' por puntos para convertirla en un número decimal válido
df = df.with_columns(pl.col("Monto").cast(pl.Float64)) # Convertir la columna 'Monto' a tipo de datos Float64 para realizar operaciones numéricas
#vamos a convertir la columna fecha a formato date
df = df.with_columns(pl.col("Fecha de factura").str.to_datetime("%m/%d/%Y %H:%M:%S"))
df = df.with_columns(pl.col("País").str.replace(" ", "_")) # Reemplazar los espacios en la columna 'País' por guiones bajos para mantener consistencia en el formato a la hora de guardar en parquet
```
**Explorar los valores nulls**
Para ver que columnas tienen valores nulos se ejecuta la siguiente celda:
<br>
```python
df.null_count()# Contar el número total de valores nulos en el DataFrame
```
<br> En nuestro dataset solo se ven valores nulos en la columna ID Cliente
```python
Valores_nullos = df['ID Cliente'].null_count()# Contar el número de valores nulos en la columna 'ID Cliente'
Total = df['ID Cliente'].count()# Contar el número total de valores no nulos en la columna 'ID Cliente'
PorcentajeNulos = (Valores_nullos / Total) * 100 # Calcular el porcentaje de valores nulos respecto al total de valores en la columna 'ID Cliente'
print("El porcentaje de ID nulos son:", PorcentajeNulos) # Imprimir el porcentaje de valores nulos en la columna 'ID Cliente'
```
<br>
Ya que para nuestro posterior ejerciciop de machine learning no es necesaria la columna ID Cliente procederemos a eliminarla, pero lo haremos **No** sobre el DF original si no sobre otro df que llamaremos **df_l**

```python
df_l=df.drop("ID Cliente")
df_l
```
<br>
Posteriormente remplazaremos los valores Nan con 0

```python
df_l.fill_nan(0)
```
# Guardar en Formato Parquet
Un argumento sólido para guardar en formato Parquet y utilizar PyArrow como motor de escritura es la optimización del rendimiento y la interoperabilidad con otros sistemas de Big Data.

1. **Eficiencia de almacenamiento:** Parquet es un formato de archivo columnar altamente eficiente que comprime y almacena los datos de manera óptima, lo que resulta en un menor consumo de espacio en disco y tiempos de lectura más rápidos. Esto es especialmente beneficioso cuando se manejan grandes volúmenes de datos.<br>
2. **Compatibilidad con sistemas de Big Data:** Parquet es ampliamente compatible con muchos sistemas de Big Data, como Apache Spark, Apache Hadoop, y otros. Al guardar los datos en formato Parquet, se asegura la interoperabilidad con estos sistemas, lo que facilita el procesamiento distribuido y la ejecución de consultas analíticas a escala.<br>
3. **PyArrow como motor de escritura:** PyArrow es una biblioteca de Python que proporciona una interfaz de alto rendimiento para interactuar con el formato Parquet. Utilizar PyArrow como motor de escritura garantiza una escritura eficiente y optimizada de los datos en archivos Parquet, aprovechando las optimizaciones internas de PyArrow y su capacidad para interactuar directamente con código C/C++ de alta velocidad.

```python
# Importar la clase Path del módulo pathlib
from pathlib import Path

# Definir la ruta del archivo utilizando la clase Path
path: Path = Path("/content/Data") / "Facturas.parquet"

# Escribir el DataFrame en formato Parquet en la ubicación especificada por 'path'
# Además, se especifica que se deben utilizar las opciones de PyArrow y se particiona por la columna 'País'
df_l.write_parquet(path, use_pyarrow=True, pyarrow_options={"partition_cols":["País"]})
```
## Si deseas descargar el parquet para utilizarlo en otro entorno o maquina:<br>
```python
# Importar las bibliotecas necesarias
import zipfile
import os

# Ruta de la carpeta que deseas comprimir
carpeta_a_comprimir = path

# Ruta del archivo ZIP resultante
archivo_zip = '/content/Parquet/partitioned_object.zip'

# Comprimir la carpeta en un archivo ZIP
with zipfile.ZipFile(archivo_zip, 'w', zipfile.ZIP_DEFLATED) as zipf:
    # Recorrer todos los archivos en la carpeta y subcarpetas
    for root, _, files in os.walk(carpeta_a_comprimir):
        for archivo in files:
            # Escribir cada archivo en el archivo ZIP
            zipf.write(os.path.join(root, archivo), os.path.relpath(os.path.join(root, archivo), carpeta_a_comprimir))

# Descargar el archivo ZIP comprimido
from google.colab import files
files.download(archivo_zip)
```
