# Apache Spark Fundamentals
## Introduction
Apache Spark is a distributed computing framework optimized for speed, ease of use, and sophisticated analytics. It is widely used for batch processing, stream processing, and machine learning tasks.

## Spark Basics
### Why Spark – Vertical vs. Horizontal Scaling
- **Vertical Scaling**: Adding more CPU, memory, and storage to a single machine (expensive but simple).
- **Horizontal Scaling**: Adding multiple machines (parallel processing, cost-effective, complex).

### What Spark is Good For
- Batch processing of large datasets
- Stream processing
- Message queue integration (Apache Kafka)
- Processing data on Hadoop

### Input Types
- CSV, JSON, Parquet
- Hadoop, ODBC, REST Calls, S3, Cloud Storage
- Streaming from Kafka, Kinesis

### Memory vs Storage
Spark leverages memory over disk storage, making it significantly faster than traditional Hadoop processing.

### Spark Architecture

![fig1 - architecture]()

- **Driver Program & Spark Context**: Main entry point, manages tasks
- **Cluster Manager**: Manages resources (YARN, Kubernetes, Standalone, Mesos)
- **Executors**: Run tasks on worker nodes
- **Tasks & Caching**: Execute transformations and store interim results in memory

### Cluster Types
- **Standalone**: Basic Spark cluster manager
- **Hadoop YARN**: Main resource manager for Hadoop clusters
- **Kubernetes**: Deploy containerized Spark applications
- **Apache Mesos**: General-purpose cluster manager

### Client vs Cluster Deployment
- **Client Mode**: The driver runs on the client machine (job stops if the client shuts down).
- **Cluster Mode**: The driver runs inside the cluster, independent of the client.

### Where to Run Spark
- Standalone
- Hadoop
- AWS EMR/Glue
- Google Cloud Dataproc
- Azure HDInsight

## Course Data & Environment
### Tools Used
- **Jupyter Docker Container**: Local Spark environment with UI for interactive development.
- Datasets Used:

  ![fig2 - contents]()

  - `bookcontents.json`
  - `bookcontents.csv`
  - `bookcontentsNoHeader.csv`
  - `sections.csv`
  - `sections_wordcount.csv`

### Docker Setup
1. Install Docker
2. Pull the Spark container
3. Start the container and mount local directories
4. Map required ports

## Spark Coding Basics
### RDDs (Resilient Distributed Datasets)
- Low-level Spark abstraction
- No automatic optimization
- Functions: `map()`, `reduce()`, `foreachpartition()`

### DataFrames
- High-level abstraction like RDBMS tables
- Schema-based, immutable, optimized

**Example: JSON Transformations**
```python
from pyspark.sql import SparkSession
spark = SparkSession.builder.appName("JSONExample").getOrCreate()
df = spark.read.json("bookcontents.json")
df.show()
```
**Output:**
```
+------+--------+----------+
|  id  | title  |  content |
+------+--------+----------+
|  1   | Spark  |  Intro   |
+------+--------+----------+
```

## Transformations & Actions
### Transformations
Transformations create new DataFrames and are evaluated lazily.
- `.select("column")` – Select specific columns
- `.where("condition")` – Filter rows
- `.join(df2, "key")` – Join DataFrames
- `.orderBy("column")` – Sort rows

### Actions
Actions execute computations and return results.
- `.write` – Writes DataFrame to storage
- `.count()`, `.first()`, `.take(n)`, `.show()`, `.collect()` – Return data to the driver

**Example: CSV Schemas**
```python
df = spark.read.option("header", True).csv("sections.csv")
df.printSchema()
df.show(5)
```

### Working with Dataframes
```python
df = spark.read.csv("sections_wordcount.csv", header=True, inferSchema=True)
df.createOrReplaceTempView("sections")
spark.sql("SELECT * FROM sections WHERE word_count > 1000").show()
```

### SparkSQL
SparkSQL allows querying structured data with SQL-like syntax.
```python
spark.sql("SELECT title, content FROM bookcontents WHERE id < 10").show()
```

### Working with RDDs
```python
rdd = spark.sparkContext.textFile("bookcontents.csv")
words = rdd.flatMap(lambda line: line.split(" "))
print(words.take(10))
```

### Notebooks for Reference
- [`01_JSON_Transformations`]()
- [`02_CSV_Schemas`]()
- [`03_Working_with_DataFrames`]()
- [`04_SparkSQL`]()
- [`05_Working_With_RDDs`]()
