---
title: "Option 2: Without Glow"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b>6.4. </b>"
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

## Set Spark Configuration

{{% notice info %}}
This section sets up the Spark configuration to use Amazon S3 Tables, AWS Glue, and Apache Iceberg.
{{% /notice %}}

If you encounter memory-related errors or are prompted to restart the kernel after executing the configuration code, it's recommended to restart the EMR Serverless Application as follows:

1. Select 'Serverless' > 'Applications' from the left menu on the EMR Studio screen.
2. Select 'workshop_emr_application' and click the 'Stop application' button to stop it.
3. Then click the 'Start application' button to start the EMR Serverless Application.
4. Verify that the cluster is properly attached in the JupyterLab notebook before continuing your work.

Access the previously created EMR Workspace (JupyterLab).

1. Create a new Notebook with the Pyspark Kernel.

2. Add the following Configuration code to the first Cell of the new Notebook:

```python
%%configure -f
{
    "conf": {
        "spark.jars": "/usr/share/aws/iceberg/lib/iceberg-spark3-runtime.jar,s3://${bucket}/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar",
        "spark.sql.catalog.s3table": "org.apache.iceberg.spark.SparkCatalog",
        "spark.sql.catalog.s3table.catalog-impl": "software.amazon.s3tables.iceberg.S3TablesCatalog",
        "spark.sql.catalog.s3table.warehouse": "${table_bucket_arn}",
        "spark.sql.extensions": "org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions",
        "spark.hadoop.io.compression.codecs": "io.projectglow.sql.util.BGZFCodec"
    }
}
```

This code configures Spark to use the following:
- Catalog settings for using AWS Glue and Amazon S3 Tables
- Adding libraries for using Apache Iceberg

The `${bucket}` and `${table_bucket_arn}` values should be changed to match your environment. Especially, change the `${table_bucket_arn}` value to the ARN value of the Table bucket you copied earlier.

3. Click the Run button to execute the notebook Cell.

![Spark configuration](/images/workshop/lab3-2-configure.png)

You are now ready to use Pyspark code referencing Amazon S3 Tables, Glue, and Apache Iceberg.

## Load VCF Data

{{% notice info %}}
This section loads VCF data using Spark.
{{% /notice %}}

1. Click the '+' button to add a Cell.

2. Read the VCF data from 1000genomes:

```python
from pyspark.sql.types import StructType, StringType 

vcf_file = "s3://1000genomes/phase1/analysis_results/integrated_call_sets/ALL.chr17.integrated_phase1_v3.20101123.snps_indels_svs.genotypes.vcf.gz"

vcf_raw_schema = StructType() \
    .add("chrom", StringType()) \
    .add("pos", StringType()) \
    .add("id", StringType()) \
    .add("ref", StringType()) \
    .add("alt", StringType()) \
    .add("qual", StringType()) \
    .add("filter", StringType()) \
    .add("info", StringType()) \
    .add("format", StringType()) \
    .add("sample", StringType())

vcf_df = spark.read \
    .option('delimiter', '\t') \
    .schema(vcf_raw_schema) \
    .csv(vcf_file)
```

Unlike Glow, this loads the VCF file by specifying the schema. It reads the VCF file with a schema consisting of chrom, pos, id, ref, alt, qual, filter, info, format, and sample columns. It reads the file as a CSV file with `\t` as the delimiter.

3. Click the '+' button to add a Cell.

4. Check the VCF file data with the following code:

```python
vcf_df.show(5, False)
```

Call the `show()` function to check 5 rows of data.

![Load VCF file](/images/workshop/lab3-2-load-vcf-data.webp)

You have loaded the VCF file from S3 into a Spark Dataframe.

## Get VCF body data and process

{{% notice info %}}
This section transforms the loaded VCF data into a more convenient form for analysis.
{{% /notice %}}

1. Click the '+' button to add a Cell.

2. Add the following code to the new Cell and 'Run':

```python
sample_id = vcf_df \
    .filter(vcf_df.chrom.startswith("#CHROM")) \
    .selectExpr('sample') \
    .collect()[0]['sample']

sample_id
```

This code retrieves the sample_id value from the VCF file.

3. Click the '+' button to add a Cell.

4. Add the following code to the new Cell and 'Run':

```python
from pyspark.sql.functions import struct, split, arrays_zip, map_from_entries, collect_list, col, lit

vcf_df = vcf_df \
    .filter(~(vcf_df.chrom.startswith("#"))) \
    .withColumn("sample_id", lit(sample_id)) \
    .withColumn("formats", split(col("format"), ':')) \
    .withColumn("samples", split(col("sample"), ':'))

vcf_df = vcf_df \
    .withColumn('zipped', arrays_zip(col('formats'), col('samples'))) \
    .withColumn('genomic', map_from_entries(col('zipped'))) \
    .select('sample_id', 'chrom', 'pos', 'id', 'ref', 
            'alt', 'qual', 'filter', 'info', 'genomic')
            
vcf_df = vcf_df \
    .groupBy("chrom", "pos", "id", "ref", "alt", "qual", "filter", "info") \
    .agg(collect_list(struct('sample_id', 'genomic')).alias("genomics"))
```

This code filters rows where the chrom column does not start with '#' to get VCF body data. The format and sample columns contain string values that can be split by ':'. These split values have a 1:1 mapping. This code transforms the dataframe to achieve this 1:1 mapping.

5. Click the '+' button to add a Cell.

6. Add the following code to the new Cell and 'Run':

```python
vcf_df.show(5, False)
vcf_df.count()
```

Call the `show()` function to check the actual transformed data. Call the `count()` function to check the row count of the processed data. It should output **1046733** rows.

![Get VCF body data](/images/workshop/lab3-2-get-vcf-data.webp)

You have transformed the Spark Dataframe loaded from the VCF file to obtain the body data.

## Create VCF Data Table

{{% notice info %}}
This section creates a table in S3 Tables from the read VCF data.
{{% /notice %}}

1. Click the '+' button to add a Cell.

2. Add the following code to the new Cell:

```python
catalog = "s3table"
namespace = "vcf"
table = "vcf_delta_2"

vcf_df \
    .write \
    .saveAsTable(f"{catalog}.{namespace}.{table}")
```

This code calls the `saveAsTable()` function to create a `vcf_delta_2` table from the previously read VCF data.

3. Click the 'Run' button to create the table with the VCF data.

4. To check the data in the created table, click the '+' button to add a Cell.

5. Add the following code to the new Cell and 'Run':

```python
s3tables_df = spark.read \
    .format("iceberg") \
    .load(f"{catalog}.{namespace}.{table}")

s3tables_df.count()
```

Call the `count()` function to check the data row count. You should see **1046733** rows, which is the same as the original file's row count.

![Create VCF table](/images/workshop/lab3-2-create-vcf-table.webp)

You have completed creating a table in S3 Tables.

From this point on, you can perform analysis queries by reading data into a Spark Dataframe, just as you did with Glow.
