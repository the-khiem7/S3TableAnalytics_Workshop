---
title: "Create S3 Tables"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b>4.3. </b>"
---

## Access EMR workspace and open notebook

{{% notice info %}}
This is the part about accessing the EMR Workspace that was set up in 'Prerequisites > Setup EMR Workspace'.
{{% /notice %}}

1. Navigate to the [EMR Console](https://console.aws.amazon.com/emr/home) page.

2. Select 'EMR Serverless' from the left menu.

3. In the right 'EMR Studio' section, select 'workshop_emr_studio' and click the 'Manage applications' button.

4. After clicking this button, a new window with the EMR Studio page will open.

5. Click on 'Workspaces' in the left menu of EMR Studio.

6. Click on the previously created 'workshop-workspace'.

7. A new window with the JupyterLab page will open.

8. Click the '+' button and select 'Notebook > PySpark' to create a new notebook.

9. In the created notebook, click the '+' button to add a Cell.

## Set Spark Configuration

{{% notice info %}}
This is the part about setting up Spark configuration to use Amazon S3 Tables, AWS Glue, and Apache Iceberg.
{{% /notice %}}

1. To check the ARN value of the previously created Table Bucket, open a new browser window and access the [S3 Console](https://console.aws.amazon.com/s3/home) page.

2. Select Table buckets from the left menu.

3. Copy the ARN (Amazon Resource Name) value of the created Table bucket.

![Table bucket ARN](/images/workshop/table_bucket_arn.webp)

{{% notice info %}}
There are two ways to configure the catalog in the Configuration code.

(Option-1) is the method of setting up S3TablesCatalog,

(Option-2) sets up a REST-based catalog.

(Note) Option-2 method does not yet support creating tables from Dataframes.

Using df.write.saveAsTable(...) will result in a `Stage-create is currently not supported.` error.

In this method, you need to create a table with spark.sql("CREATE TABLE ...") and then insert data with df.write.InsertInto(...).

Both methods work, but for convenience in this lab, we will proceed with the (Option-1) method.
{{% /notice %}}

4. Click the '+' button to create a new Cell and add the following Configuration code.

**(Option-1)** This is the method of configuring with S3TablesCatalog.

```python
%%configure -f
{
    "conf": {
        "spark.jars": "/usr/share/aws/iceberg/lib/iceberg-spark3-runtime.jar,s3://<bucket>/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar",
        "spark.sql.defaultCatalog": "s3table",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.catalog-impl": "software.amazon.s3tables.iceberg.S3TablesCatalog",
        "spark.sql.catalog.s3table.warehouse": "<table_bucket_arn>",
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.driver.extraJavaOptions": "-Dfile.encoding=UTF-8",
        "spark.executor.extraJavaOptions": "-Dfile.encoding=UTF-8"                
    }
}        
```

- The `<bucket>` value should be replaced with the name of your S3 bucket.
- The `<table_bucket_arn>` value should be replaced with the ARN of the Table Bucket you created earlier.
- `spark.driver.extraJavaOptions` and `spark.executor.extraJavaOptions` were added to prevent potential issues with Korean character encoding.

**(Option-2)** This is the method for configuring using the REST interface of Amazon S3 Tables.

```python
%%configure -f
{
    "conf": {
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.sql.defaultCatalog": "s3table",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.type": "rest",
        "spark.sql.catalog.s3table.uri": "https://s3tables.<region>.amazonaws.com/iceberg",
        "spark.sql.catalog.s3table.warehouse": "<table_bucket_arn>",
        "spark.sql.catalog.s3table.rest.sigv4-enabled": "true",
        "spark.sql.catalog.s3table.rest.signing-name": "s3tables",
        "spark.sql.catalog.s3table.rest.signing-region": "<region>",
        "spark.sql.catalog.s3table.io-impl": "org.apache.iceberg.aws.s3.S3FileIO",
        "spark.sql.catalog.s3table.rest-metrics-reporting-enabled": "false",
        "spark.driver.extraJavaOptions": "-Dfile.encoding=UTF-8",
        "spark.executor.extraJavaOptions": "-Dfile.encoding=UTF-8"                
    }
}
```

- The code above configures Spark to enable the following:
- Setting up a REST-type catalog for using Amazon S3 Tables
- Adding the necessary library for using Apache Iceberg
- Replace the `<table_bucket_arn>` value with the ARN of the Table Bucket created earlier.
- Set `<region>` value to 'ap-northeast-2' or 'us-east-1'.

5. Click the Run button to execute the notebook cell.

You're now ready to use PySpark code with Amazon S3 Tables, AWS Glue, and Apache Iceberg.

{{% notice warning %}}
If you encounter memory-related errors or kernel restart prompts after running the configuration code and then executing Spark code, it is recommended to restart the EMR Serverless Application as follows:

1. In the EMR Studio left-hand menu, go to 'Serverless' > 'Applications'.
2. Select 'workshop_emr_application', then click the 'Stop application' button to stop it.
3. After that, click the 'Start application' button to start the EMR Serverless Application.
4. In your JupyterLab notebook, ensure the cluster is properly attached, then continue your work.
{{% /notice %}}

## Read CSV Data from S3 and Create Tables

{{% notice info %}}
This section describes how to use Spark to read CSV data from S3 and create a table.
{{% /notice %}}

1. Access the EMR Workspace (JupyterLab) that you created earlier.

2. Click the '+' button, then select 'Notebook > PySpark' to create a new notebook.

3. In the newly created notebook, click the '+' button again to add a new cell.

4. Copy and paste the code below to set the required variables for the task, and update `<bucket>` value to match your environment.

```python
datasets = [
    "allergies",
    "careplans",
    "conditions",
    "devices",
    "encounters",
    "imaging_studies",
    "immunizations",
    "medications",
    "observations",
    "organizations",
    "patients",
    "payer_transitions",
    "payers",
    "procedures",
    "providers"
]

bucket = "<bucket>"
catalog = "s3table"
namespace = "workshop_namespace"
```

- Replace `<bucket>` with the appropriate S3 bucket name for your environment.

5. Click the Run button to execute the notebook cell.

6. Click the '+' button to add a new cell.

7. Paste the code below into the newly added cell.

```python
for dataset in datasets:
    df = spark.read.option("header", True).csv(f"s3://{bucket}/data/coherent/{dataset}.csv")
    df = df.toDF(*[col.lower() for col in df.columns])
    df.write \
        .saveAsTable(f"{catalog}.{namespace}.{dataset}")     
```

- The code above reads 15 CSV files uploaded to S3 into a DataFrame, then creates an Iceberg table in Amazon S3 Tables.
- It loops through the 15 datasets and sequentially creates tables.

{{% notice info %}}
When creating tables for actual production use, it's recommended to specify partition columns as shown below.

Data is stored in a hierarchy based on the partition columns.

This structure allows data pruning based on partition columns during queries, which significantly impacts query performance in the future.

For now, we won't specify partitions to facilitate a smooth hands-on experience. (This is just for your reference)
{{% /notice %}}

```python
# from pyspark.sql.functions import dayofmonth, substring, col

# df = df.withColumn("day", dayofmonth("reg_date")) \
#     .withColumn("id_prefix", substring(col("id").cast("string"), 1, 2))

# df.write \
#     .partitionBy("day", "id_prefix") \
#     .saveAsTable("database_name.table_name")    
```

8. Click the Run button to execute the notebook cell.

The CSV files in S3 have been loaded into Spark DataFrames, and the tables have been successfully created.

## (Optional) Check Tables

{{% notice info %}}
Let's check whether the tables created earlier have been properly created.
{{% /notice %}}

1. Click the '+' button to add a new cell.

2. Copy and paste the code below into the newly added cell.

```sql
%%sql
SHOW TABLES FROM s3table.workshop_namespace
```

3. Click the Run button to execute the notebook cell.

4. If the tables are displayed as shown below, the process was completed successfully.

![Table check](/images/workshop/table-check.webp)

5. Run the queries below to check the data within the tables.

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.patients
```

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.observations
```

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.conditions
```

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.medications
```

The CSV data in S3 has been successfully read into a Spark DataFrame and created as Iceberg tables.
