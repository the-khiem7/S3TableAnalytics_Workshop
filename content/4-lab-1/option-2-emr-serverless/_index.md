---
title: "Option 2: EMR Serverless"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b>5.4. </b>"
---

## Access EMR workspace and open notebook

{{% notice info %}}
This section covers accessing the EMR Workspace set up in Prerequisites > Setup EMR Workspace.
{{% /notice %}}

Go to the [EMR Console](https://console.aws.amazon.com/emr) page.

Select 'EMR Serverless' from the left menu.

In the 'EMR Studio' section on the right, select 'workshop_emr_studio' and click the 'Manage applications' button.

Clicking this button will open the EMR Studio page in a new window.

Click 'Workspaces' in the left menu of EMR Studio.

Click on the previously created 'workshop-workspace'.

A new window with the JupyterLab page will open.

In the left menu of JupyterLab, click to open the 'workshop-workspace.ipynb' notebook that has been created.

Select 'Pyspark' in the 'Select Kernel' option.

![Open notebook](/images/workshop/emr-open-notebook.png)

You are now ready to create Namespace and Table in S3 Tables based on EMR Serverless.

## Set Spark configuration

{{% notice info %}}
This section sets up Spark configuration to use Amazon S3 Tables, AWS Glue, and Apache Iceberg.
{{% /notice %}}

To check the ARN value of the previously created Table Bucket, open a new browser window and access the [S3 Console](https://console.aws.amazon.com/s3) page.

Select Table buckets from the left menu.

Copy the ARN (Amazon Resource Name) value of the created Table bucket.

![Table bucket ARN](/images/workshop/table-bucket-arn.png)

Click the '+' button to create a new Cell and add the following Configuration code:

**(Option-1)** This is the method to set up S3TablesCatalog.

```python
%%configure -f
{
    "conf": {
        "spark.jars": "/usr/share/aws/iceberg/lib/iceberg-spark3-runtime.jar,s3://<bucket>/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.catalog-impl": "software.amazon.s3tables.iceberg.S3TablesCatalog",
        "spark.sql.catalog.s3table.warehouse": "<table_bucket_arn>",
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.driver.extraJavaOptions": "-Dfile.encoding=UTF-8",
        "spark.executor.extraJavaOptions": "-Dfile.encoding=UTF-8"                
    }
}        
```

- The `bucket` value should be changed to your S3 bucket name.
- The `table_bucket_arn` value should be changed to the ARN value of the Table Bucket you created earlier.
- `spark.driver.extraJavaOptions` and `spark.executor.extraJavaOptions` are added to prevent potential Korean character encoding issues.

**(Option-2)** This is the method to set up Amazon S3 Tables using the REST approach.

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

This code configures Spark to use the following:
- REST type catalog configuration for using Amazon S3 Tables
- Adding libraries for using Apache Iceberg
- The `table_bucket_arn` value should be changed to the ARN value of the Table Bucket you created earlier.
- Enter 'ap-northeast-2' or 'us-east-1' for the region value.

{{% notice info %}}
There are two ways to set up the catalog in the Configuration code. (Option-1) is the method to set up S3TablesCatalog, and (Option-2) sets up a REST-style catalog. Both methods work, but for convenience, we will proceed with (Option-1) in this lab.

(Note) Option-2 method does not yet support creating tables from Dataframes. Using `df.write.saveAsTable(...)` will result in a `Stage-create is currently not supported.` error. In this method, you need to create the table using `spark.sql("CREATE TABLE ...")` and then insert data using `df.write.InsertInto(...)`.
{{% /notice %}}

Click the Run button to execute the notebook Cell.

You are now ready to use Pyspark code referencing Amazon S3 Tables, Glue, and Apache Iceberg.

## Read CSV Data from S3

{{% notice info %}}
This section covers reading CSV data from S3 using Spark. If you encounter memory-related errors or are prompted to restart the kernel after executing the configuration code and then running Spark code, it's recommended to restart the EMR Serverless Application as follows: Select 'Serverless' > 'Applications' from the left menu on the EMR Studio screen. Select 'workshop_emr_application' and click the 'Stop application' button to stop it. Then click the 'Start application' button to start the EMR Serverless Application. Verify that the cluster is properly attached in the JupyterLab notebook before continuing your work.
{{% /notice %}}

Access the previously created EMR Workspace (JupyterLab).

Open 'workshop_workspace.ipynb'.

Click the '+' button to add a Cell.

Read the CSV data uploaded to S3:

```python
from pyspark.sql.types import StructType, StringType, DoubleType, DateType

bucket = '<bucket>'

schema = StructType() \
    .add("id", StringType()) \
    .add("legal_dong_code", StringType()) \
    .add("legal_dong_name", StringType()) \
    .add("category_code", StringType()) \
    .add("category_name", StringType()) \
    .add("land_number", StringType()) \
    .add("reference_year", StringType()) \
    .add("refernece_month", StringType()) \
    .add("disclosure_price", DoubleType()) \
    .add("disclosure_date", DateType()) \
    .add("is_standard", StringType()) \
    .add("base_date", DateType()) \
    .add("sido_code", StringType())

df = spark.read \
    .option("header", "true") \
    .schema(schema) \
    .csv(f"s3://{bucket}/data/AL_D151_11_20240125_UTF8.csv")
```

- Replace `<bucket>` with the appropriate S3 bucket name for your environment.
- Define the schema and then read the CSV data from S3.

![Read CSV file](/images/workshop/emr-read-csv-data.png)

Click the '+' button to add a Cell.

Check the CSV file data with the following code:

```python
df.show(5, False)
```

Call the `show()` function to display 5 rows of data.

Click the '+' button to add a Cell.

Check the number of rows in the CSV data with the following code:

```python
df.count()
```

Call the `count()` function to output the number of rows.
You should see that there are 897963 rows.

The CSV file from S3 has been loaded into a Spark DataFrame.

## Create Data Table

{{% notice info %}}
This section covers creating a table in S3 Tables from the loaded CSV data.
{{% /notice %}}

Click the '+' button to add a Cell.

Add the following code to the new Cell:

```python
catalog = "s3table"
namespace = "workshop_namespace"
table = "individually_disclosed_building_price_by_emr"

df.write \
    .option("encoding", "UTF-8") \
    .saveAsTable(f"{catalog}.{namespace}.{table}") 
```

This code calls the `saveAsTable()` function to create a table named `individually_disclosed_building_price_by_emr` from the previously loaded CSV data.

Click the 'Run' button to create the table from the CSV data.

![Create table](/images/workshop/emr-create-table.webp)

To check the data in the created table, click the '+' button to add a Cell.

Add the following code to the new Cell and 'Run' it:

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.count()
```

Call the `count()` function to check the number of data rows.
You should see that there are 897963 rows, the same as in the original file.

The table has been successfully created in S3 Tables.

## Merge CSV Data

{{% notice info %}}
This section demonstrates performing a merge query.
{{% /notice %}}

Click the '+' button to add a Cell.

Add the following code to the new Cell and 'Run' it:

```python
other_df = spark.read \
    .option("header", "true") \
    .schema(schema) \
    .csv(f"s3://{bucket}/data/AL_D151_41_20250120_200000_UTF8.csv")

other_df.createOrReplaceTempView("other_source_table")
```

- Read another CSV file data (AL_D151_41_20250120_200000_UTF8.csv)
- This file combines public land price data of 100,000 records from Seoul and 100,000 records from Gyeonggi-do.
- In other words, it contains a total of 200,000 records, of which 100,000 are duplicates of the data previously inserted into S3 Tables.
- `other_df.createOrReplaceTempView("other_source_table")` — This part registers the DataFrame loaded from the CSV file as a Temp View. Once registered, it can be referenced like a table in Spark SQL statements.

Click the '+' button to add a Cell.

Add the following code to the new Cell and 'Run' it:

```python
spark.sql(f"""
    MERGE INTO {catalog}.{namespace}.{table} t
    USING other_source_table s
    ON (s.id=t.id and s.legal_dong_code=t.legal_dong_code and 
        s.legal_dong_name=t.legal_dong_name and s.category_code=t.category_code and 
        s.category_name=t.category_name and s.land_number=t.land_number and 
        s.reference_year=t.reference_year and s.refernece_month=t.refernece_month and 
        s.disclosure_price=t.disclosure_price and s.disclosure_date=t.disclosure_date and 
        s.is_standard=t.is_standard and s.base_date=t.base_date and s.sido_code=t.sido_code)
    WHEN NOT MATCHED THEN
    INSERT (id, legal_dong_code, legal_dong_name, 
            category_code, category_name, land_number, 
            reference_year, refernece_month, 
            disclosure_price, disclosure_date, 
            is_standard, base_date, sido_code)
    VALUES (s.id, s.legal_dong_code, s.legal_dong_name, 
            s.category_code, s.category_name, s.land_number, 
            s.reference_year, s.refernece_month, 
            s.disclosure_price, s.disclosure_date, 
            s.is_standard, s.base_date, s.sido_code)
""")
```

This executes a Merge query using `spark.sql()`.
In the Merge query:
- `{catalog}.{namespace}.{table}` is the Target table for the merge.
- `other_source_table` is the Source table registered as a Temp View earlier.
- The ON clause sets the matching condition. In this query, it checks if all columns are the same.
- `WHEN NOT MATCHED THEN` followed by the INSERT statement is executed for data that doesn't match the condition. This means new data will be inserted into the target table.

![Merge data](/images/workshop/emr-merge-data.webp)

Click the '+' button to add a Cell.

Add the following code to the new Cell and 'Run' it:

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.count()
```

- Reload the table.
- Check the number of rows in the table.
- As previously explained, the data used for Merge contains a total of 200,000 records, of which 100,000 are already loaded in the table.
- Therefore, the row count should increase by only 100,000 records.
- Confirmed that 897963 + 100000 becomes 997963 rows, verifying that the Merge was completed successfully.

We have successfully merged the same CSV data into the previously created table. If you have time, you can try uploading a different CSV file to S3 and merging it.

## Update / Delete Data

{{% notice info %}}
This section covers updating or deleting data in the S3 Tables table.
{{% /notice %}}

### Update Rows

Click the '+' button to add a Cell.

Add the following code to the new Cell and 'Run' it:

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.filter("legal_dong_code = '1111010100'").count()
s3tables_df.filter("legal_dong_code = '1111010100'").show(5, False)
```

This calls `filter("legal_dong_code = '1111010100'")` to count the rows where 'legal_dong_code' is '1111010100'.
You should see that there are 821 rows.

Click the '+' button to add a Cell.

Add the following code to the new Cell and 'Run' it:

```python
spark.sql(f"""
UPDATE {catalog}.{namespace}.{table} 
SET legal_dong_name = 'Seoul Jongno-gu Cheongunbong-dong' 
WHERE legal_dong_code = '1111010100'
""")
```

This updates the `legal_dong_name` to 'Seoul Jongno-gu Cheongunbong-dong' for rows where `legal_dong_code` is '1111010100'.

![Update table](/images/workshop/emr-update-table.png)

Click the '+' button to add a Cell.

Add the following code to the new Cell and 'Run' it:

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.filter("legal_dong_code = '1111010100'").show(5, False)
```

- This reloads the table.
- It then checks the rows where the `legal_dong_code` column value is '1111010100'.
- You should see that the value has been changed to 'Seoul Jongno-gu Cheongunbong-dong'.

The update has been completed successfully.

### Delete Rows

{{% notice warning %}}
Make note of the time when you perform the delete query. This will be used in the snapshot-related exercise later.
{{% /notice %}}

Click the '+' button to add a Cell.

Add the following code to the new Cell and 'Run' it:

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.filter("is_standard = 'Y'").count()
s3tables_df.filter("is_standard = 'Y'").show(5, False)
```

This calls `filter("is_standard = 'Y'")` to count the rows where 'is_standard' is 'Y'.
You should see that there are 30861 rows.

Click the '+' button to add a Cell.

Add the following code to the new Cell and 'Run' it:

```python
spark.sql(f"""
DELETE FROM {catalog}.{namespace}.{table}
WHERE is_standard = 'Y'
""")
```

This executes a DELETE query for rows where the `is_standard` column value is 'Y'.

![Delete table](/images/workshop/emr-delete-table.webp)

Click the '+' button to add a Cell.

Insert the following code into the added Cell and 'Run':

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.filter("is_standard = 'Y'").count()
```

- This reloads the table.
- It checks the Row count of data where the `is_standard` column value is 'Y'.
- Confirm that 0 is output as all have been deleted.

The deletion has been completed successfully.

## Time Travel

### Get Table History (Snapshots)

```python
spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}.history
""").show(5, False)
```

```
+-----------------------+-------------------+-------------------+-------------------+
|made_current_at        |snapshot_id        |parent_id          |is_current_ancestor|
+-----------------------+-------------------+-------------------+-------------------+
|2025-04-02 02:26:45.356|1996392549628528971|NULL               |true               |
|2025-04-02 02:27:16.224|2842948084683911314|1996392549628528971|true               |
|2025-04-02 02:27:37.664|4907549559143542142|2842948084683911314|true               |
|2025-04-02 02:28:08.795|5291307565494698844|4907549559143542142|true               |
+-----------------------+-------------------+-------------------+-------------------+
```

This outputs the Snapshot history of the table.
If you executed in the order of the workshop,
- The top Row is the Snapshot when the table was created/Inserted
- The second is the Snapshot when the Merge query was performed
- The third is the Snapshot when the Update query was performed
- The fourth is the Snapshot when the Delete query was performed.

### Get Snapshot Information

```python
snapshot_id = "<snapshot_id>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}.snapshots
WHERE snapshot_id = {snapshot_id} 
""").show(5, False)
```

This outputs the metadata file location and file-related data for the Snapshot.

### Select Rows with Update Snapshot

```python
update_snapshot_id = "<update_snapshot_id>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table} 
FOR VERSION AS OF {update_snapshot_id}
WHERE legal_dong_code = '1111010100'
""").show(5, False)
```

- Set the `<snapshot_id>` value to the Snapshot ID value from before the update.
- You can confirm that the data that was updated to 'Seoul Jongno-gu Cheongun-dong' is output with the value from before the update.

### Select Rows before Deletion Time

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}
FOR TIMESTAMP AS OF TIMESTAMP '{deletion_time}'
WHERE is_standard = 'Y'
""").show(5, False)
```

- Set `deletion_time` to a time before the data deletion.
- You can check the Rows that were deleted.

### Deletion Rollback

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
CALL {catalog}.system.rollback_to_timestamp('{namespace}.{table}', TIMESTAMP '{deletion_time}')
""")

spark.sql(f"""
SELECT COUNT(*) FROM {catalog}.{namespace}.{table}
WHERE is_standard = 'Y'
""").show()
```

- Set `deletion_time` to a time before the data deletion.
- You can confirm that the previously deleted data has been rolled back.

We have examined the Time Travel queries of Apache Iceberg together.
