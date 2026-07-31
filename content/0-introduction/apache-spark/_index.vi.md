---
title: "Apache Spark"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>1.2. </b>"
---

{{% notice info %}}
Apache Spark là một framework xử lý phân tán mã nguồn mở cho xử lý dữ liệu quy mô lớn, cho phép xử lý dữ liệu nhanh chóng và linh hoạt. Tôi sẽ mô tả các khái niệm cốt lõi của Spark dưới dạng danh sách.
{{% /notice %}}

## 1. Tổng quan Apache Spark

Apache Spark là một công cụ xử lý dữ liệu phân tán được thiết kế để xử lý lượng lớn dữ liệu một cách nhanh chóng. Nó cung cấp tốc độ nhanh hơn tới 100 lần so với Hadoop MapReduce truyền thống và hỗ trợ các thao tác dựa trên bộ nhớ, cho phép phân tích dữ liệu hiệu quả.

Spark hoạt động chủ yếu trong môi trường cluster và có thể xử lý dữ liệu song song trên nhiều node. Điều này cho phép nó thực hiện các workload dữ liệu lớn đa dạng như kỹ thuật dữ liệu, học máy và xử lý luồng.

## 2. Các tính năng chính

Các tính năng chính của Apache Spark như sau:

- Tốc độ nhanh: Spark hỗ trợ tính toán trong bộ nhớ, giảm thiểu I/O đĩa cho các thao tác dữ liệu lặp lại.
- Mô hình tính toán linh hoạt: Spark hỗ trợ nhiều thao tác bao gồm xử lý batch, xử lý dữ liệu streaming, tính toán đồ thị và học máy.
- Hỗ trợ nhiều ngôn ngữ: Hỗ trợ các ngôn ngữ như Scala, Java, Python và R, cho phép các nhà phát triển phát triển ứng dụng Spark trong môi trường quen thuộc.
- Hệ sinh thái phong phú: Spark cung cấp nhiều thư viện tích hợp sẵn, giúp dễ dàng phát triển các mô hình phân tích dữ liệu và học máy.

## 3. Các thành phần cốt lõi của Spark

Spark bao gồm nhiều thành phần cốt lõi, mỗi thành phần có một vai trò cụ thể.

Trong workshop này, chúng ta sẽ sử dụng **Spark SQL**.

- Spark Core: Công cụ cơ bản của Spark thực hiện tính toán phân tán, chịu trách nhiệm lập lịch tác vụ và quản lý bộ nhớ.
- Spark SQL: Một module xử lý dữ liệu dựa trên SQL có thể tích hợp với nhiều kho dữ liệu bao gồm RDBMS hiện có và Hive, cũng như các loại tệp khác nhau và lưu trữ đối tượng như S3.
- Spark Streaming: Một thành phần xử lý dữ liệu thời gian thực có thể tích hợp với các nguồn dữ liệu streaming như Kafka, Flume và Kinesis.
- MLlib (Machine Learning Library): Một thư viện hỗ trợ phát triển mô hình học máy dễ dàng, tối ưu hóa các thao tác ML trong môi trường phân tán.
- GraphX: Một thành phần cho các thao tác dữ liệu đồ thị, có khả năng thực hiện các thuật toán đồ thị như PageRank.

## 4. RDD (Resilient Distributed Dataset)

Cấu trúc dữ liệu cốt lõi của Spark là RDD (Resilient Distributed Dataset).

RDD là cấu trúc dữ liệu phân tán cơ bản được cung cấp bởi Spark, được thiết kế để ngăn mất dữ liệu và xử lý dữ liệu song song trên nhiều node trong cluster.

Các đặc điểm chính của RDD như sau:

- Tính bất biến: Một khi RDD được tạo, nó không thể thay đổi. Các thao tác tạo ra RDD mới.
- Xử lý phân tán: Dữ liệu có thể được xử lý hiệu quả trên nhiều node cho các thao tác quy mô lớn.
- Khả năng chịu lỗi: Khi mất dữ liệu xảy ra, RDD có thể được tạo lại dựa trên thông tin lineage.

## 5. DataFrame

DataFrame của Spark là một tập hợp dữ liệu phân tán có cấu trúc tương tự như bảng SQL, bao gồm các hàng và cột.

Không giống RDD, DataFrame bao gồm schema và có thể tối ưu hóa các thao tác dựa trên cột.

Chúng cũng có thể dễ dàng tích hợp với nhiều nguồn dữ liệu khác nhau (Hive, Parquet, JSON, CSV, JDBC, v.v.).

### 5.1. Các tính năng chính của DataFrame

- Xử lý dữ liệu có cấu trúc
  Không giống RDD, dữ liệu được lưu trữ theo cột và có thể xử lý như SQL.
- Hỗ trợ SQL và nhiều ngôn ngữ
  Có thể sử dụng truy vấn SQL, và DataFrame có thể được xử lý bằng các ngôn ngữ như Scala, Python, Java và R.
- Tích hợp với nhiều nguồn dữ liệu
  Hỗ trợ các định dạng tệp như JSON, CSV, Parquet, ORC, Avro, v.v.
  Có thể tích hợp với các nguồn dữ liệu như JDBC, Hive, HDFS và S3.

## 6. Cấu trúc thực thi Spark

Ứng dụng Spark có cấu trúc thực thi như sau:

1. Driver: Thực thi ứng dụng Spark do người dùng viết và quản lý toàn bộ công việc.
2. Cluster Manager: Quản lý tài nguyên cluster để chạy ứng dụng Spark. (ví dụ: YARN, Kubernetes, Mesos)
3. Executor: Các tiến trình worker thực sự thực thi các tác vụ trong cluster.
4. Task: Đơn vị công việc thực tế được thực thi trên một Executor.

Khi một ứng dụng Spark được thực thi, Driver yêu cầu công việc từ Cluster Manager, sau đó phân phối các tác vụ đến các Executor khác nhau để thực thi.

## 7. Mã PySpark chính

Dưới đây là một số ví dụ sử dụng DataFrame trong PySpark mà chúng ta sẽ sử dụng trong workshop.

### 7.1. Tạo SparkSession

Tất cả thao tác PySpark bắt đầu với việc tạo `SparkSession`. Tuy nhiên, trong workshop này, chúng ta sử dụng môi trường notebook JupyterLab nơi SparkSession đã được tạo sẵn, vì vậy chúng ta không cần sử dụng đoạn mã bên dưới.

```python
from pyspark.sql import SparkSession

# Create SparkSession
spark = SparkSession.builder.appName("PySparkExample").getOrCreate()
```

### 7.2. Tạo DataFrame

#### 7.2.1. Đọc từ tệp CSV

```python
df = spark.read.csv("data.csv", header=True, inferSchema=True)
df.show()
```

#### 7.2.2. Đọc từ tệp JSON

```python
df = spark.read.json("data.json")
df.show()
```

#### 7.2.3. Đọc từ tệp Parquet

```python
df = spark.read.parquet("data.parquet")
df.show()
```

### 7.3. Chọn cột và lọc

#### 7.3.1. Chọn các cột cụ thể

```python
df.select("name", "age").show()
```

#### 7.3.2. Lọc theo điều kiện

```python
df.filter(df.age > 25).show()
```

#### 7.3.3. Lọc dữ liệu chứa giá trị cụ thể

```python
df.filter(df.name.startswith("A")).show()
```

#### 7.3.4. Lọc với nhiều điều kiện

```python
df.filter((df.age > 25) & (df.gender == "Female")).show()
```

### 7.4 Chuyển đổi dữ liệu và các thao tác bổ sung

#### 7.4.1. Thêm cột mới

```python
from pyspark.sql.functions import col

df = df.withColumn("age_plus_5", col("age") + 5)
df.show()
```

Kết quả đầu ra:

```
+---------+---+------+-----------+
|     name|age|gender|age_plus_5 |
+---------+---+------+-----------+
|   Alice| 25|Female|        30 |
|     Bob| 30|  Male|        35 |
|Catherine| 27|Female|        32 |
+---------+---+------+-----------+
```

#### 7.4.2. Đổi tên cột

```python
df = df.withColumnRenamed("age", "years_old")
df.show()
```

#### 7.4.3. Xóa cột

```python
df = df.drop("age_plus_5")
df.show()
```

### 7.5. Các thao tác nhóm và tổng hợp

#### 7.5.1. Nhóm và đếm

```python
df.groupBy("gender").count().show()
```

#### 7.5.2. Tính tuổi trung bình

```python
df.groupBy("gender").avg("age").show()
```

#### 7.5.3. Áp dụng nhiều thao tác tổng hợp

```python
from pyspark.sql.functions import avg, max, min

df.groupBy("gender").agg(avg("age").alias("avg_age"), max("age").alias("max_age")).show()
```

### 7.6. Sắp xếp

#### 7.6.1. Sắp xếp theo tuổi tăng dần

```python
df.orderBy("age").show()
```

#### 7.6.2. Sắp xếp theo tuổi giảm dần

```python
df.orderBy(col("age").desc()).show()
```

### 7.7. SQL

Spark DataFrame có thể thực thi SQL trực tiếp.

```python
df.createOrReplaceTempView("people")

spark.sql("SELECT name, age FROM people WHERE age > 25").show()
```

### 7.8. Kết hợp dữ liệu (Join & Union)

#### 7.8.1. Kết hợp hai DataFrame (Join)

```python
data1 = [("Alice", "HR"), ("Bob", "Finance"), ("Catherine", "IT")]
df1 = spark.createDataFrame(data1, ["name", "department"])

data2 = [("Alice", 5000), ("Bob", 6000), ("Catherine", 7000)]
df2 = spark.createDataFrame(data2, ["name", "salary"])

df_joined = df1.join(df2, "name", "inner")
df_joined.show()
```

Kết quả đầu ra:

```
+---------+----------+------+
|     name|department|salary|
+---------+----------+------+
|   Alice|       HR |  5000|
|     Bob|  Finance |  6000|
|Catherine|      IT |  7000|
+---------+----------+------+
```

#### 7.8.2. Gộp hai DataFrame (Union)

```python
df1 = spark.createDataFrame([("Alice", 25), ("Bob", 30)], ["name", "age"])
df2 = spark.createDataFrame([("Charlie", 35), ("David", 40)], ["name", "age"])

df_union = df1.union(df2)
df_union.show()
```

### 7.9. Lưu trữ

#### 7.9.1. Lưu dưới dạng tệp CSV

```python
df.write.csv("output.csv", header=True)
```

#### 7.9.2. Lưu dưới dạng tệp Parquet

```python
df.write.parquet("output.parquet")
```

#### 7.9.3. Lưu dưới dạng tệp JSON

```python
df.write.json("output.json")
```
