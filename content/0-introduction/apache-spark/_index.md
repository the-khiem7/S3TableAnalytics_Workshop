---
title: "Apache Spark"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>1.2. </b>"
---

{{% notice info %}}
Apache Spark is an open-source distributed processing framework for large-scale data processing, enabling fast and flexible data processing. I will describe the core concepts of Spark in a list format.
{{% /notice %}}

## 1. Apache Spark Overview

Apache Spark is a distributed data processing engine designed to process large amounts of data quickly. It provides speeds up to 100 times faster than traditional Hadoop MapReduce and supports memory-based operations, allowing for efficient data analysis.

Spark operates primarily in a cluster environment and can process data in parallel across multiple nodes. This enables it to perform various big data workloads such as data engineering, machine learning, and stream processing.

## 2. Key Features

The main features of Apache Spark are as follows:

- Fast speed: Spark supports in-memory computing, minimizing disk I/O for repetitive data operations.
- Flexible computation model: Spark supports various operations including batch processing, streaming data processing, graph computations, and machine learning.
- Multiple language support: It supports languages such as Scala, Java, Python, and R, allowing developers to develop Spark applications in familiar environments.
- Rich ecosystem: Spark provides various built-in libraries, making it easy to develop data analysis and machine learning models.

## 3. Spark's Core Components

Spark consists of several core components, each with a specific role.

In this workshop, we will be using **Spark SQL**.

- Spark Core: The basic engine of Spark that performs distributed computing, responsible for task scheduling and memory management.
- Spark SQL: A module for SQL-based data processing that can integrate with various data stores including existing RDBMSs and Hive, as well as different file types and object storage like S3.
- Spark Streaming: A component for real-time data processing that can integrate with streaming data sources such as Kafka, Flume, and Kinesis.
- MLlib (Machine Learning Library): A library that supports easy development of machine learning models, optimizing ML operations in distributed environments.
- GraphX: A component for graph data operations, capable of performing graph algorithms like PageRank.

## 4. RDD (Resilient Distributed Dataset)

The core data structure of Spark is the RDD (Resilient Distributed Dataset).

RDD is the basic distributed data structure provided by Spark, designed to prevent data loss and process data in parallel across multiple nodes in a cluster.

The main characteristics of RDD are as follows:

- Immutability: Once an RDD is created, it cannot be changed. Operations create new RDDs.
- Distributed Processing: Data can be processed efficiently across multiple nodes for large-scale operations.
- Fault Tolerance: When data loss occurs, RDDs can be recreated based on lineage information.

## 5. DataFrame

Spark's DataFrame is a distributed data collection structured similarly to SQL tables, composed of rows and columns.

Unlike RDDs, DataFrames include a schema and can optimize column-based operations.

They can also easily integrate with various data sources (Hive, Parquet, JSON, CSV, JDBC, etc.).

### 5.1. Key Features of DataFrame

- Structured Data Processing
  Unlike RDDs, data is stored by column and can be handled like SQL.
- SQL and Multiple Language Support
  SQL queries can be used, and DataFrames can be handled in languages like Scala, Python, Java, and R.
- Integration with Various Data Sources
  Supports file formats like JSON, CSV, Parquet, ORC, Avro, etc.
  Can integrate with data sources like JDBC, Hive, HDFS, and S3.

## 6. Spark Execution Structure

Spark applications have the following execution structure:

1. Driver: Executes the Spark application written by the user and manages the overall job.
2. Cluster Manager: Manages cluster resources to run Spark applications. (e.g., YARN, Kubernetes, Mesos)
3. Executor: Worker processes that actually execute tasks within the cluster.
4. Task: The actual unit of work executed on an Executor.

When a Spark application is executed, the Driver requests work from the Cluster Manager, which then distributes tasks to various Executors for execution.

## 7. Key PySpark Code

Here are some examples of using DataFrames in PySpark that we'll be using in the workshop.

### 7.1. Creating a SparkSession

All PySpark operations start with creating a `SparkSession`. However, in this workshop, we're using a JupyterLab notebook environment where a SparkSession is already created, so we don't need to use the code below.

```python
from pyspark.sql import SparkSession

# Create SparkSession
spark = SparkSession.builder.appName("PySparkExample").getOrCreate()
```

### 7.2. Creating DataFrames

#### 7.2.1. Reading from a CSV file

```python
df = spark.read.csv("data.csv", header=True, inferSchema=True)
df.show()
```

#### 7.2.2. Reading from a JSON file

```python
df = spark.read.json("data.json")
df.show()
```

#### 7.2.3. Reading from a Parquet file

```python
df = spark.read.parquet("data.parquet")
df.show()
```

### 7.3. Column Selection and Filtering

#### 7.3.1. Selecting specific columns

```python
df.select("name", "age").show()
```

#### 7.3.2. Condition filtering

```python
df.filter(df.age > 25).show()
```

#### 7.3.3. Filtering data containing specific values

```python
df.filter(df.name.startswith("A")).show()
```

#### 7.3.4. Filtering with multiple conditions

```python
df.filter((df.age > 25) & (df.gender == "Female")).show()
```

### 7.4 Data Transformation and Additional Operations

#### 7.4.1. Adding a new column

```python
from pyspark.sql.functions import col

df = df.withColumn("age_plus_5", col("age") + 5)
df.show()
```

Output result:

```
+---------+---+------+-----------+
|     name|age|gender|age_plus_5 |
+---------+---+------+-----------+
|   Alice| 25|Female|        30 |
|     Bob| 30|  Male|        35 |
|Catherine| 27|Female|        32 |
+---------+---+------+-----------+
```

#### 7.4.2. Renaming a column

```python
df = df.withColumnRenamed("age", "years_old")
df.show()
```

#### 7.4.3. Dropping a column

```python
df = df.drop("age_plus_5")
df.show()
```

### 7.5. Grouping and Aggregation Operations

#### 7.5.1. Grouping and counting

```python
df.groupBy("gender").count().show()
```

#### 7.5.2. Calculating average age

```python
df.groupBy("gender").avg("age").show()
```

#### 7.5.3. Applying multiple aggregation operations

```python
from pyspark.sql.functions import avg, max, min

df.groupBy("gender").agg(avg("age").alias("avg_age"), max("age").alias("max_age")).show()
```

### 7.6. Sorting

#### 7.6.1. Sorting by age in ascending order

```python
df.orderBy("age").show()
```

#### 7.6.2. Sorting by age in descending order

```python
df.orderBy(col("age").desc()).show()
```

### 7.7. SQL

Spark DataFrames can directly execute SQL.

```python
df.createOrReplaceTempView("people")

spark.sql("SELECT name, age FROM people WHERE age > 25").show()
```

### 7.8. Combining Data (Join & Union)

#### 7.8.1. Combining two DataFrames (Join)

```python
data1 = [("Alice", "HR"), ("Bob", "Finance"), ("Catherine", "IT")]
df1 = spark.createDataFrame(data1, ["name", "department"])

data2 = [("Alice", 5000), ("Bob", 6000), ("Catherine", 7000)]
df2 = spark.createDataFrame(data2, ["name", "salary"])

df_joined = df1.join(df2, "name", "inner")
df_joined.show()
```

Output result:

```
+---------+----------+------+
|     name|department|salary|
+---------+----------+------+
|   Alice|       HR |  5000|
|     Bob|  Finance |  6000|
|Catherine|      IT |  7000|
+---------+----------+------+
```

#### 7.8.2. Merging two DataFrames (Union)

```python
df1 = spark.createDataFrame([("Alice", 25), ("Bob", 30)], ["name", "age"])
df2 = spark.createDataFrame([("Charlie", 35), ("David", 40)], ["name", "age"])

df_union = df1.union(df2)
df_union.show()
```

### 7.9. Saving

#### 7.9.1. Saving as a CSV file

```python
df.write.csv("output.csv", header=True)
```

#### 7.9.2. Saving as a Parquet file

```python
df.write.parquet("output.parquet")
```

#### 7.9.3. Saving as a JSON file

```python
df.write.json("output.json")
```
