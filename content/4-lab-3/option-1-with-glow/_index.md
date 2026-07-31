---
title: "Option 1: with Glow"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b>6.3. </b>"
---

## Access EMR workspace and open notebook

{{% notice info %}}
This section covers accessing the EMR Workspace set up in Prerequisites > Setup EMR Workspace.
{{% /notice %}}

1. Go to the [EMR Console](https://console.aws.amazon.com/emr/) page.

2. Select 'EMR Serverless' from the left menu.

3. In the 'EMR Studio' section on the right, select 'workshop_emr_studio' and click the 'Manage applications' button.

4. Clicking this button will open the EMR Studio page in a new window.

5. Click 'Workspaces' in the left menu of EMR Studio.

6. Click on the previously created 'workshop-workspace'.

7. A new window with the JupyterLab page will open.

8. In the left menu of JupyterLab, click to open the 'workshop-workspace.ipynb' notebook that has been created.

9. Select 'Pyspark' in the 'Select Kernel' option.

![Open notebook](/images/workshop/emr-open-notebook.png)

You are now ready to create Namespace and Table in S3 Tables based on EMR Serverless.

## Set Spark Configuration for Glow

{{% notice info %}}
This section sets up the Spark configuration to use a Python 3.10.12 virtual environment packaged with Amazon S3 Tables, AWS Glue, Apache Iceberg, and Glow.
{{% /notice %}}

If you encounter memory-related errors or are prompted to restart the kernel when running Spark code after executing the configuration code, it's recommended to restart the EMR Serverless Application as follows:

1. Select 'Serverless' > 'Applications' from the left menu on the EMR Studio screen.
2. Select 'workshop_emr_application' and click the 'Stop application' button to stop it.
3. Then click the 'Start application' button to start the EMR Serverless Application.
4. Verify that the cluster is properly attached in the JupyterLab notebook before continuing your work.

Access the previously created EMR Workspace (JupyterLab).

1. Open 'workshop_workspace.ipynb'.

2. Add the following configuration code to the first cell of the new Notebook:

```python
%%configure -f
{
    "conf": {
        "spark.jars": "/usr/share/aws/iceberg/lib/iceberg-spark3-runtime.jar,s3://${bucket}/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar,s3://${bucket}/lib/glow-spark3-assembly-2.0.0.jar",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.catalog-impl": "software.amazon.s3tables.iceberg.S3TablesCatalog",
        "spark.sql.catalog.s3table.warehouse": "${table_bucket_arn}",
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.hadoop.io.compression.codecs": "io.projectglow.sql.util.BGZFCodec",
        "spark.archives": "s3://${bucket}/lib/pyspark_venv_python_3.10.12.tar.gz#environment",
        "spark.executorEnv.PYSPARK_PYTHON": "./environment/bin/python",
        "spark.emr-serverless.driverEnv.PYSPARK_DRIVER_PYTHON": "./environment/bin/python",
        "spark.emr-serverless.driverEnv.PYSPARK_PYTHON": "./environment/bin/python"
    }
}
```

This code configures Spark to use the following:
- Catalog settings for using AWS Glue and Amazon S3 Tables
- Adding libraries for using Apache Iceberg
- Python 3.10.12 virtual environment settings for using Glow with the above two

The following configuration values correspond to this:
- `spark.archives`
- `spark.executorEnv.PYSPARK_PYTHON`
- `spark.emr-serverless.driverEnv.PYSPARK_DRIVER_PYTHON`
- `spark.emr-serverless.driverEnv.PYSPARK_PYTHON`

The `${bucket}` and `${table_bucket_arn}` values should be changed to match your environment. Particularly, change the `${table_bucket_arn}` value to the ARN of the Table bucket you copied earlier.

3. Click the Run button to execute the notebook cell.

![Spark configuration](/images/workshop/lab3-configure.png)

Preparations are now complete to use Pyspark code referencing Amazon S3 Tables, Glue, Apache Iceberg, as well as the Glow library.

## Load VCF Data

{{% notice info %}}
This section loads VCF data using Glow and Spark.
{{% /notice %}}

1. Click the '+' button to add a cell.

2. Add the following code to the new cell and run it to start the Spark App:

```python
import glow
spark = glow.register(spark)
```

This code registers the Spark session with Glow. This enables Glow-based data reading, transformation, and related functions in Spark.

3. Read the VCF data from 1000genomes:

```python
vcf_file = "s3://1000genomes/phase1/analysis_results/integrated_call_sets/ALL.chr17.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.vcf.gz"

df = spark.read.format("vcf").load(vcf_file)
df.printSchema()
```

`spark.read.format("vcf").load(vcf_file)` — As you can see from this code, Spark loads the VCF file in 'vcf' format.

Click the 'Run' button to execute this cell. The `printSchema()` function call will output the schema of the VCF data.

![Load VCF file](/images/workshop/lab3-load-vcf-data.webp)

4. Click the '+' button to add a cell.

5. Use the following code to determine the row count of the VCF file:

```python
df.count()
```

This will output **1046733**. This is the row count of the VCF file.

We have loaded the VCF file from S3 into a Spark Dataframe and output the number of rows in that Dataframe.

## Create VCF Data Table

{{% notice info %}}
This section creates a table in S3 Tables from the loaded VCF data.
{{% /notice %}}

1. Click the '+' button to add a cell.

2. Add the following code to the new cell:

```python
catalog = "s3table"
namespace = "workshop_namespace"
table = "vcf_delta"

df = df.toDF(*[col.lower() for col in df.columns])

df \
    .write \
    .saveAsTable(f"{catalog}.{namespace}.{table}")
```

This code calls the `saveAsTable()` function. It creates a `vcf_delta` table from the previously loaded VCF data.

`df = df.toDF(*[col.lower() for col in df.columns])` — If column names in the df are uppercase, they may not be properly recognized in Athena. Therefore, all column names in the df are changed to lowercase.

3. Click the 'Run' button to create the table with this VCF data.

![Create VCF table](/images/workshop/lab3-create-vcf-table.webp)

4. To check the data in the created table, click the '+' button to add a cell.

5. Add the following code to the new cell:

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.drop('genotypes').show(3, False)
```

This declares a Dataframe that reads the `vcf_delta` table. It calls the `show()` function to display 3 rows of data on the screen.

Run this cell and check the output to confirm that the table was created correctly.

6. Click the '+' button to add a cell.

7. Add the following code to the new cell and run it:

```python
vcf_df.count()
```

This calls the `count()` function to check the data row count. You can confirm that it outputs **1046733** rows, the same as the original file's row count.

We have completed creating a table in S3 Tables using the Spark Dataframe loaded from the VCF file.

## Merge another VCF Data

{{% notice info %}}
This section reads another VCF data and merges it into the table in S3 Tables.
{{% /notice %}}

1. Click the '+' button to add a cell.

2. Add the following code to the new cell and run it:

```python
other_vcf_file = "s3://1000genomes/phase1/analysis_results/integrated_call_sets/ALL.chr18.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.vcf.gz"

other_df = spark.read.format("vcf").load(other_vcf_file)
other_df.count()
```

This reads another VCF file. It calls the `count()` function to check the row count of this VCF data. It outputs **1088820** rows.

3. Click the '+' button to add a cell.

4. Add the following code to the new cell and run it:

```python
other_df = other_df.toDF(*[col.lower() for col in other_df.columns])
other_df.createOrReplaceTempView("other_vcf_table")
```

This changes all column names in the Dataframe to lowercase, as done previously. This registers the Dataframe that read the other VCF file data as a Temp View. Once registered like this, it can be referenced like a table in Spark SQL statements.

5. Click the '+' button to add a cell.

6. Add the following code to the new cell and run it:

```python
spark.sql(f"""
    MERGE INTO {catalog}.{namespace}.{table} t
    USING other_vcf_table s
    ON (s.contigname=t.contigname and s.start=t.start and s.end=t.end and s.names=t.names and 
        s.referenceallele=t.referenceallele and s.alternatealleles=t.alternatealleles)
    WHEN MATCHED THEN
    UPDATE SET
        t.contigname = s.contigname,
        t.start = s.start,
        t.end = s.end,
        t.names = s.names,
        t.referenceallele = s.referenceallele,
        t.alternatealleles = s.alternatealleles,
        t.qual = s.qual,
        t.filters = s.filters,
        t.splitfrommultiallelic = s.splitfrommultiallelic,
        t.info_avgpost = s.info_avgpost,
        t.info_ac = s.info_ac,
        t.info_ciend = s.info_ciend,
        t.info_ldaf = s.info_ldaf,
        t.info_afr_af = s.info_afr_af,
        t.info_vt = s.info_vt,
        t.info_snpsource = s.info_snpsource,
        t.info_an = s.info_an,
        t.info_theta = s.info_theta,
        t.info_cipos = s.info_cipos,
        t.info_aa = s.info_aa,
        t.info_af = s.info_af,
        t.info_amr_af = s.info_amr_af,
        t.info_asn_af = s.info_asn_af,
        t.info_svlen = s.info_svlen,
        t.info_erate = s.info_erate,
        t.info_homseq = s.info_homseq,
        t.info_rsq = s.info_rsq,
        t.info_end = s.info_end,
        t.info_eur_af = s.info_eur_af,
        t.info_homlen = s.info_homlen,
        t.info_svtype = s.info_svtype
    WHEN NOT MATCHED THEN
    INSERT (contigname, start, end, names, referenceallele, alternatealleles, 
            qual, filters, splitfrommultiallelic, info_avgpost, 
            info_ac, info_ciend, info_ldaf, info_afr_af, info_vt, info_snpsource, info_an, info_theta, 
            info_cipos, info_aa, info_af, info_amr_af, info_asn_af, info_svlen, info_erate, 
            info_homseq, info_rsq, info_end, info_eur_af, info_homlen, info_svtype)
    VALUES (s.contigname, s.start, s.end, s.names, s.referenceallele, s.alternatealleles, 
            s.qual, s.filters, s.splitfrommultiallelic, s.info_avgpost, 
            s.info_ac, s.info_ciend, s.info_ldaf, s.info_afr_af, s.info_vt, s.info_snpsource, s.info_an, s.info_theta, 
            s.info_cipos, s.info_aa, s.info_af, s.info_amr_af, s.info_asn_af, s.info_svlen, s.info_erate, 
            s.info_homseq, s.info_rsq, s.info_end, s.info_eur_af, s.info_homlen, s.info_svtype)
""")
```

This executes a Merge query using `spark.sql()`.

Looking at the Merge query:
- `{catalog}.{namespace}.{table}` is the Target table to merge data into.
- `other_vcf_table` is the Source table registered earlier as a Temp View.
- The `ON` clause sets the matching condition.
- `WHEN MATCHED THEN` is followed by the UPDATE statement that executes for matching data.
- `WHEN NOT MATCHED THEN` is followed by the INSERT statement that executes for non-matching data.
- In other words, it UPDATEs the target table for existing data and INSERTs into the target table for new data.

![Merge VCF data](/images/workshop/lab3-merge-vcf-data.webp)

7. Click the '+' button to add a cell.

8. Add the following code to the new cell and run it:

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.count()
```

This reloads the table. It checks the Row Count of the table. Since there are no matching rows between the two merged Dataframes, it outputs **2135553**, which is the sum of the 1046733 rows initially entered when creating the table and the 1088820 rows entered this time. This confirms that the merge was successful.

We have completed merging other VCF data into the previously created table.

## Update / Delete VCF Data

{{% notice info %}}
This section updates or deletes data in the S3 Tables table.
{{% /notice %}}

### Update Rows

1. Click the '+' button to add a cell.

2. Add the following code to the new cell and run it:

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.filter("qual < 100").count()
```

Call `filter("qual < 100")` to query the row count of data with quality less than 100. You can confirm there are **10382** records.

3. Click the '+' button to add a Cell.

4. Put the code below in the added Cell and click 'Run'.

```python
vcf_df \
    .filter("qual < 100") \
    .drop('genotypes') \
    .show(5, False)
```

Shows 5 records with quality less than 100. Will batch update the qual values of this data to 0.

5. Click the '+' button to add a Cell.

6. Put the code below in the added Cell and click 'Run'.

```python
spark.sql(f"""
UPDATE {catalog}.{namespace}.{table} SET qual = 0 WHERE qual < 100
""")
```

Batch update qual values to 0 for data where the qual column value is less than 100.

![Update VCF table](/images/workshop/lab3-update-vcf-table.webp)

7. Click the '+' button to add a Cell.

8. Put the code below in the added Cell and click 'Run'.

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.filter("qual = 0").count()
```

Reload the table. Check the row count where qual column value is 0. Confirm that the same value of **10382** records as before the update is displayed.

Update completed successfully.

### Delete Rows

1. Click the '+' button to add a Cell.

2. Insert the following code into the added Cell and 'Run':

```python
spark.sql(f"""
DELETE FROM {catalog}.{namespace}.{table} WHERE qual = 0
""")
```

This performs a DELETE query for data where the qual column value is 0.

![Delete VCF table](/images/workshop/lab3-delete-vcf-table.webp)

3. Click the '+' button to add a Cell.

4. Insert the following code into the added Cell and 'Run':

```python
vcf_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

vcf_df.filter("qual = 0").count()
```

This reloads the table. It checks the Row count of data where the qual column value is 0. Confirm that 0 is output as all have been deleted.

The deletion has been completed successfully.

## Time Travel

### Get Table History (Snapshots)

```python
spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}.history
""").show(5, False)
```

```text
+-----------------------+-------------------+-------------------+-------------------+
|made_current_at        |snapshot_id        |parent_id          |is_current_ancestor|
+-----------------------+-------------------+-------------------+-------------------+
|2025-04-02 02:26:45.356|1996392549628528971|NULL               |true               |
|2025-04-02 02:27:16.224|2842948084683911314|1996392549628528971|true               |
|2025-04-02 02:27:37.664|4907549559143542142|2842948084683911314|true               |
|2025-04-02 02:28:08.795|5291307565494698844|4907549559143542142|true               |
+-----------------------+-------------------+-------------------+-------------------+
```

This outputs the Snapshot history of the table. If you executed in the order of the workshop:
- The top Row is the Snapshot when the table was created/Inserted
- The second is the Snapshot when the Merge query was performed
- The third is the Snapshot when the Update query was performed
- The fourth is the Snapshot when the Delete query was performed

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
WHERE qual = 0
""").show(5, False)
```

Set the `<snapshot_id>` value to the Snapshot ID value from before the update. You can confirm that the data that was updated is output with the value from before the update.

### Select Rows before Deletion Time

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
SELECT * FROM {catalog}.{namespace}.{table}
FOR TIMESTAMP AS OF TIMESTAMP '{deletion_time}'
WHERE qual = 0
""").show(5, False)
```

Set `deletion_time` to a time before the data deletion. You can check the Rows that were deleted.

### Deletion Rollback

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
CALL {catalog}.system.rollback_to_timestamp('{namespace}.{table}', TIMESTAMP '{deletion_time}')
""")

spark.sql(f"""
SELECT COUNT(*) FROM {catalog}.{namespace}.{table}
WHERE qual = 0
""").show()
```

Set `deletion_time` to a time before the data deletion. You can confirm that the data deleted earlier has been rolled back.

We have reviewed Apache Iceberg's Time Travel queries together.
