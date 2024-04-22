# Polars: A Glimpse into Tabular Data Processing Efficiency

**What is Polars?**
Polars is a tabular data processing library in Rust, designed to offer exceptional performance and an easy-to-use API. Although relatively new compared to more established libraries like Pandas in Python, Polars has quickly gained popularity due to its speed and efficiency in processing large volumes of data.

1. **Superior Performance:**
One of the main advantages of Polars is its superior performance compared to Pandas. This is largely due to Polars being written in Rust, a programming language known for its speed and safety. Polars' efficient implementation allows for significantly faster data processing operations than Pandas, making it ideal for applications requiring fast processing of large datasets.

2. **Advanced Features:**
Polars offers a wide range of advanced features for tabular data processing. This includes filtering, grouping, joining, and transforming data, as well as the ability to handle missing data efficiently. Additionally, Polars is compatible with parallelism and distribution operations, allowing for the full utilization of available hardware resources to further accelerate data processing.

3. **Integration with Python:**
Although Polars is written in Rust, it provides a Python interface that allows users to leverage all its features directly from the Python environment. This facilitates the integration of Polars into existing data analysis workflows in Python, enabling users to take advantage of its superior performance without having to learn a new programming language.

In summary, Polars is a tabular data processing library that stands out for its superior performance and advanced features. Whether you are working with small or large datasets, Polars offers an efficient and scalable solution for data processing in Python environments. If you are looking to improve the performance of your data analysis applications, Polars is definitely a tool worth exploring.

## Data
* In this scenario, we are working with a dataset extracted from Kaggle, a well-known repository of datasets. The goal is to explore and prepare this data for subsequent storage in Parquet format. This format is known for its efficiency in processing large volumes of data, especially when working with Apache-based Big Data systems, such as Apache Spark.<br>
By using Parquet, we are taking advantage of the benefits offered by Apache in Big Data processing. Parquet is a highly efficient columnar file format that stores data in columns rather than rows. This structure is highly efficient for analytical queries and processing operations in distributed systems. Additionally, Parquet offers advanced data compression and encoding, reducing file sizes and further improving performance in distributed environments.<br>
When exploring and preparing the data, we are performing a series of technical steps, such as data cleaning, data transformation, filtering, and aggregation. These steps are crucial to ensure the quality and consistency of the data before further processing.

**Install Polars**
```python
!pip install polars
```
<br>
If you are working in Google Colab, I want to inform you that polars is already installed in the execution environment
<br>
Remember, before reading the data, look for the access path where you have saved the csv file

```python
df=pl.read_csv('/content/ventas-por-factura.csv')#Read the path
df.head()
```
**Necessary Transformations** <br>
In my case, I made some necessary data type transformations to have a good schema in the data

```python
df = df.with_columns(pl.col("Monto").cast(pl.Utf8)) # Convert the 'Monto' column to Utf8 data type to make it manipulable
df = df.with_columns(pl.col("Monto").str.replace(",", ".")) # Replace commas in the 'Monto' column with periods to make it a valid decimal number
df = df.with_columns(pl.col("Monto").cast(pl.Float64)) # Convert the 'Monto' column to Float64 data type for numerical operations
# Let's convert the date column to date format
df = df.with_columns(pl.col("Fecha de factura").str.to_datetime("%m/%d/%Y %H:%M:%S"))
df = df.with_columns(pl.col("País").str.replace(" ", "_")) # Replace spaces in the 'País' column with underscores to maintain consistency in the format when saving in parquet
```
**Explore Null Values**

To see which columns have null values, execute the following cell:
<br>
```python
df.null_count()# Count the total number of null values in the DataFrame
```
<br> In our dataset, only null values are seen in the ID Cliente column
```python
Valores_nullos = df['ID Cliente'].null_count()# Count the number of null values in the 'ID Cliente' column
Total = df['ID Cliente'].count()# Count the total number of non-null values in the 'ID Cliente' column
PorcentajeNulos = (Valores_nullos / Total) * 100 # Calculate the percentage of null values relative to the total values in the 'ID Cliente' column
print("The percentage of null IDs is:", PorcentajeNulos) # Print the percentage of null values in the 'ID Cliente' column
```
<br>
Since for our subsequent machine learning exercise the ID Cliente column is not necessary, we will proceed to delete it, but we will do it **not** on the original DF but on another df that we will call **df_l**

```python
df_l=df.drop("ID Cliente")
df_l
```
Subsequently, we will replace NaN values with 0

```python
df_l.fill_nan(0)
```
# Save in Parquet Format

A compelling argument for saving in Parquet format and using PyArrow as the writing engine.

1. **Storage Efficiency:** Parquet is a highly efficient columnar file format that compresses and stores data optimally, resulting in lower disk space consumption and faster read times. This is especially beneficial when handling large volumes of data.
2. **Compatibility with Big Data Systems:** Parquet is widely compatible with many Big Data systems, such as Apache Spark, Apache Hadoop, and others. By saving data in Parquet format, interoperability with these systems is ensured, facilitating distributed processing and execution of analytical queries at scale.
3. **PyArrow as the Writing Engine:** PyArrow is a Python library that provides a high-performance interface for interacting with Parquet format. Using PyArrow as the writing engine ensures efficient and optimized writing of data to Parquet files, leveraging internal optimizations of PyArrow and its ability to interact directly with high-speed C/C++ code.

```python
# Import necessary libraries
from pathlib import Path

# Define the file path using the Path class
path: Path = Path("/content/Data") / "Facturas.parquet"

# Write the DataFrame in Parquet format at the specified location by 'path'
# Additionally, specify to use PyArrow options and partition by the 'País' column
df_l.write_parquet(path, use_pyarrow=True, pyarrow_options={"partition_cols":["País"]})
```
## If you want to download the parquet to use it in another environment or machine:

```python
# Import necessary libraries
import zipfile
import os

# Path of the folder you want to compress
carpeta_a_comprimir = path

# Path of the resulting ZIP file
archivo_zip = '/content/Parquet/partitioned_object.zip'

# Compress the folder into a ZIP file
with zipfile.ZipFile(archivo_zip, 'w', zipfile.ZIP_DEFLATED) as zipf:
    # Traverse through all files in the folder and subfolders
    for root, _, files in os.walk(carpeta_a_comprimir):
        for archivo in files:
            # Write each file into the ZIP file
            zipf.write(os.path.join(root, archivo), os.path.relpath(os.path.join(root, archivo), carpeta_a_comprimir))

# Download the compressed ZIP file
from google.colab import files
files.download(archivo_zip)

```
