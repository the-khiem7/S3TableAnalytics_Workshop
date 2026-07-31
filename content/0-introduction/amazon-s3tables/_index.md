---
title: "Amazon S3 Tables"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>1.1. </b>"
---

{{% notice info %}}
Amazon S3 Tables is a new service officially launched by AWS in late 2023. It is a table-format data service based on Amazon S3 object storage. It simplifies data lake construction and management. Built on the Apache Iceberg open-source table format, it allows data analysts and engineers to easily manage data stored in S3.
{{% /notice %}}

Before we dive into Amazon S3 Tables, let's first take a look at Apache Iceberg.

## 1. Apache Iceberg

Apache Iceberg is an open table format for managing large-scale analytical datasets.

It was initiated by Netflix and is currently managed by the Apache Software Foundation.

Iceberg addresses the limitations of the existing Hive table format and provides ACID transactions, schema evolution, partitioning optimization, and metadata management for large-scale datasets.

https://github.com/apache/iceberg

### 1.1. Key Features

- ACID Transaction Support: Maintains data consistency even in parallel operations.
- Schema Evolution: Supports adding, deleting, and renaming columns without downtime.
- Hidden Partitioning: Enables optimized queries without exposing partitioning columns.
- Time Travel: Ability to roll back or query specific points in time or specific snapshots.
- Incremental Read: Efficient as it can read only changed data.
- Pluggable Catalog Support: Can integrate with various metastores such as Hadoop, AWS Glue, Hive Metastore, etc.

### 1.2. Architecture

![Apache Iceberg Architecture](/images/workshop/apache-iceberg-architecture.webp)

#### 1.2.1. Key Components

- Iceberg Catalog
  Contains pointers to the current metadata file of the table.
  A new metadata file is created every time data changes, and the pointer points to the most recent metadata file.
  This aspect enables ACID within Iceberg tables.
- Metadata Layer
  Table Metadata (JSON file): Stores information such as table schema, location, partition information, snapshot timestamps, etc.
  Manifest List (Avro file): Metadata file pointing to the file list of a single snapshot.
  Manifest (Avro file): List of actual data files and partition/row statistics for each file.
- Data Layer
  Data Files (Parquet, ORC, Avro): Actual data files.

#### 1.2.2. Snapshot-based Structure

- All data changes are made by creating new snapshots,
- Existing snapshots are maintained as-is, making Time Travel and Rollback easy to support.

### 1.3. Detailed Features

#### 1.3.1. Schema Evolution

```sql
ALTER TABLE sales ADD COLUMN region STRING;
ALTER TABLE sales DROP COLUMN discount;
ALTER TABLE sales RENAME COLUMN price TO unit_price;
```

#### 1.3.2. Time Travel

```sql
-- Query based on a specific snapshot
SELECT * FROM sales.snapshots WHERE snapshot_id = 123456789;
SELECT count(*) FROM sales FOR TIMESTAMP AS OF TIMESTAMP '2022-01-01 00:00:00.000000 Z';
SELECT count(*) FROM sales FOR VERSION AS OF 2188465307835585443;

-- Rollback to a specific time
CALL system.rollback_to_timestamp('sales', TIMESTAMP '2024-03-01 00:00:00');
```

#### 1.3.3. Incremental Read

```sql
SELECT * FROM sales FOR CHANGES AS OF SNAPSHOT 123456789;
```

#### 1.3.4. Data Compaction

```sql
CALL system.rewrite_data_files("sales");
```

#### 1.3.5. Table Update/Delete

```sql
-- Delete statement
DELETE FROM iceberg_table WHERE category='c3';

-- Update statement
UPDATE iceberg_table SET category='c2' WHERE category='c1';

-- Merge into statement
MERGE INTO accounts t 
    USING monthly_accounts_update s
    ON (t.customer = s.customer)
    WHEN MATCHED AND s.address = 'Centreville'
        THEN DELETE
    WHEN MATCHED
        THEN UPDATE
            SET purchases = s.purchases + t.purchases, address = s.address
    WHEN NOT MATCHED
        THEN INSERT (customer, purchases, address)
              VALUES(s.customer, s.purchases, s.address);
```

### 1.4. Integration and Compatibility

#### 1.4.1. Data processing engines

- Apache Spark
- Apache Flink
- Trino / Presto
- Apache Hive
- Snowflake
- AWS Athena
- Dremio
- Starburst
- ...

#### 1.4.2. Data lake compatibility

- S3
- GCS
- Azure Blob Storage

### 1.5. Optimization

#### 1.5.1. Partitioning data

```sql
CREATE TABLE prod.db.sample (
    id bigint,
    data string,
    category string,
    ts timestamp)
USING iceberg
PARTITIONED BY (bucket(16, id), days(ts), category);
```

#### 1.5.2. Sorting data

```sql
CALL catalog_name.system.rewrite_data_files(
  table => 'db.sample', 
  strategy => 'sort', 
  sort_order => 'zorder(col_1, col_2)'
);
```

#### 1.5.3. Compacting small files

```sql
CALL catalog_name.system.rewrite_data_files('db.sample');
```

#### 1.5.4. Expiring old snapshots

```sql
ALTER TABLE prod.db.sample SET TBLPROPERTIES (
    'history.expire.max-snapshot-age-ms'='432000000',
    'history.expire.min-snapshots-to-keep'='1'
);

CALL catalog_name.system.expire_snapshots('db.sample');
```

#### 1.5.5. Removing orphaned files

```sql
CALL catalog_name.system.remove_orphan_files(table => 'db.sample');
```

#### 1.5.6. Managing snapshot retention

```sql
CREATE ICEBERG TABLE prod.db.sample()
    PARTITIONED BY $event_date
    TABLE_DATA_RETENTION = 5 days
```

#### 1.5.7. Automating optimization tasks

![Automation for optimization](/images/workshop/automation-screenshot.webp)

### 1.6. Apache Iceberg vs. Delta Lake

Both are table formats designed to support ACID in Data lakes.

#### 1.6.1. Comparison

| Feature | Apache Iceberg | Delta Lake |
| --- | --- | --- |
| Definition | Iceberg table format provides a scalable infrastructure with support for multiple processing engines. | Delta Lake is a reliable storage layer, especially suitable for the Databricks ecosystem. |
| File format | Iceberg supports various file formats, including Parquet, Avro, and ORC. | Delta Lake natively supports only the Parquet file format. |
| ACID properties | Iceberg supports ACID transactions. | Delta Lake offers robust ACID properties. |
| Partition handling | Iceberg supports dynamic partitioning, meaning partitions can be updated without rewriting the schema. | Partitions are constant, and you should define them when creating tables. Modifying defined partitions might involve data rewrites. |
| Time travel | Every change made to the table creates a new snapshot. | It offers time travel features through transaction logs, with changes tracked in JSON files. |
| Integrations | Iceberg supports multiple data processing engines, such as SQL, Spark, Trino, Hive, Flink, Presto, and more. | Delta Lake is tightly coupled with Apache Spark. |

#### 1.6.2. Use cases

| Use Case | Apache Iceberg | Delta Lake |
| --- | --- | --- |
| Engine flexibility | It's best when using multiple engines, including Apache Spark, Flink, Presto, Hive, etc. It's Ideal for environments needing different engines for different processing tasks. | Best for users who natively use Apache Spark, offering tight integration and optimal performance within the Spark ecosystem. |
| Data streaming | Supports continuous data ingestion from various sources, processing it in real time. | It unifies batch and stream processing, which is ideal for use cases requiring both in a single pipeline. |

### 1.7. Best Practices

- Manifest File Cleanup (Compaction): As data files increase, performance may degrade, so perform regular compaction.
- Careful Partition Design: Even with Hidden Partition, choose appropriate partition keys considering query performance.
- Careful Catalog Management: When using AWS Glue or Hive Metastore, proper permission and policy management is essential.
- Data Format Selection: While Iceberg supports Parquet, ORC, and Avro, Parquet is generally recommended.

## 2. Amazon S3 Tables ?

### 2.1. Background

We have been using S3 for various workloads so far.

Among the countless diverse workloads, the proportion of utilizing S3 as a tabular data store is steadily increasing.

Particularly, Parquet files are one of the most common and fastest-growing data types in S3.

In other words, use cases of uploading Parquet files to S3 based on the Apache Iceberg table format and using them through various query engines are growing rapidly.

However, to maintain and utilize Apache Iceberg-based data on S3 for analysis, users inevitably have to pay attention to many aspects.

For example,

- As the scale grows, better performance is required.
- Maintaining security at the table level is not a simple issue.
- Optimizing storage costs can become an unexpected operational burden.

Amazon S3 Tables is a managed service launched to allow users to focus on data operations without worrying about these aspects.

### 2.2. Benefits

- Scalability
  Simplifies data lakes of all sizes in an Iceberg environment.
- Enhanced performance
  Provides up to 3 times faster query performance through continuous table optimization compared to unmanaged Iceberg tables.
  Can process up to 10 times more transactions per second compared to Iceberg tables stored in general-purpose S3 buckets.
- Fully managed
  Performs ongoing table maintenance tasks such as compression, snapshot management, and removal of unreferenced files.
  Automatically optimizes query efficiency and costs over time.
- Seamless integration
  Easily integrates with AWS Analytics services like Amazon Athena, Redshift, EMR through Glue Catalog and S3 Tables integration.
  Also compatible with other popular open-source tools.
- Simplified security
  Easily manage access to tables by creating them as AWS resources and applying permissions through integration with Lake Formation.

### 2.3. Cost Structure

https://aws.amazon.com/ko/s3/pricing/

- S3 Storage: S3 storage costs for storing data and metadata
  Monitoring: USD 0.025 per 1000 objects
  Up to 50TB/month: USD 0.0265/GB
  50-450TB/month: USD 0.0265/GB
  500TB+/month: USD 0.0242/GB
  For S3 standard,
  Up to 50TB/month: USD 0.023/GB
  50-450TB/month: USD 0.022/GB
  500TB+/month: USD 0.021/GB
- API Calls: API call costs for table creation, modification, deletion, and query operations
  PUT, POST, LIST requests per 1000: USD 0.005
  GET and all other requests per 1000: USD 0.0004
- Maintenance
  Compression - USD 0.004 per 1000 objects
  Compression - USD 0.05 per GB of processed data
