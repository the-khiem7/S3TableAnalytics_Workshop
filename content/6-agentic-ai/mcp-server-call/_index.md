---
title: "Using MCP Server"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b>8.3. </b>"
---

{{% notice info %}}
We will manage Table Buckets, Namespaces, Tables, etc. using Amazon Q CLI with LLM and Amazon S3 Tables MCP server.
{{% /notice %}}

## Executing Prompts Related to Amazon S3 Tables

{{% notice warning %}}
If you see a message indicating that the Foundation Model is unavailable when executing prompts, select claude-3.7-sonnet.
{{% /notice %}}

![FM unavailable](/images/workshop/fm-unavailable.webp)

### Creating a Table Bucket

Execute the following prompt:

```text
Create a table bucket with name hello-world
```

It responds as follows:

```text
> I'll create a table bucket named hello-world for you using the S3 Tables MCP Server.


🛠️  Using tool: create_table_bucket from mcp server awslabss_3_tables_mcp_server
 ⋮ 
 ● Running create_table_bucket with the param:
 ⋮  {
 ⋮    "bucket_name": "hello-world",
 ⋮    "options": {
 ⋮      "region": "<region>"
 ⋮    }
 ⋮  }

Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:

> y


 ⋮ 
 ● Completed in 1.615s


> Perfect! I've successfully created the S3 table bucket named hello-world. 

Table Bucket Details:
• Name: hello-world
• ARN: arn:aws:s3tables:<region>:<accountId>:bucket/hello-world
• Region: <region>

The table bucket is now ready to use. You can:
• Create namespaces within this bucket to organize your tables
• Add tables to those namespaces for storing your data
• Each table bucket can hold up to 10,000 tables by default

Would you like me to help you create a namespace within this table bucket or perform any other operations?
```

- The LLM inferred from the user's prompt and then executed a Tool called `create_table_bucket`.
- The `create_table_bucket` Tool creates a Table Bucket named `hello-world` as per the user's input.
- Verify the Table Bucket creation in the S3 Console.

### Creating a Namespace

Execute the following prompt:

```text
Create a namespace named default
```

It responds as follows:

```text
> I'll create a namespace named default in the hello-world table bucket for you.


🛠️  Using tool: create_namespace from mcp server awslabss_3_tables_mcp_server
 ⋮ 
 ● Running create_namespace with the param:
 ⋮  {
 ⋮    "bucket_name": "hello-world",
 ⋮    "options": {
 ⋮      "namespace_name": "default",
 ⋮      "region": "<region>"
 ⋮    }
 ⋮  }

Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:

> y


 ⋮ 
 ● Completed in 0.784s


> Excellent! I've successfully created the default namespace in your hello-world table bucket.

Namespace Details:
• Name: default
• Bucket: hello-world
• ARN: arn:aws:s3tables:<region>:<accountId>:bucket/hello-world

The namespace is now ready for use. Namespaces provide logical groupings for your tables within the table bucket, helping you organize your 
data effectively.

You can now:
• Create tables within this default namespace
• Import data from CSV or Parquet files into tables
• Run SQL queries against tables in this namespace

Would you like me to help you create a table in this namespace or perform any other operations?
```

- The LLM inferred from the user's prompt and then executed a Tool called `create_namespace`.
- The `create_namespace` Tool creates a Namespace named `default` as per the user's input.

### Creating a Table

Execute the command below to upload a CSV file to S3.

```bash
BUCKET=$(aws s3api list-buckets --query "Buckets[0].Name" --output text)
curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/observations.csv' --output observations.csv
aws s3 cp observations.csv s3://${BUCKET}/data/coherent/
```

Execute the `q chat` command to run Amazon Q CLI, then execute the prompt below.

```text
Create a s3 table from s3://${BUCKET}/data/coherent/observations.csv in hello-world table bucket and default namespace
```

{{% notice info %}}
Replace `${BUCKET}` with your own bucket name.
{{% /notice %}}

It responds as follows:

```text
> I'll create an S3 table from your CSV file in the hello-world table bucket and default namespace. Let me import the CSV data which will automatically create the table with the appropriate schema.

🛠️  Using tool: import_csv_to_table from mcp server awslabss_3_tables_mcp_server
⋮
 ● Running import_csv_to_table with the param:
 ⋮  {
 ⋮    "bucket_name": "hello-world",
 ⋮    "options": {
 ⋮      "namespace_name": "default",
 ⋮      "table_name": "observations",
 ⋮      "source_uri": "s3://${BUCKET}/data/coherent/observations.csv",
 ⋮      "region": "<region>",
 ⋮      "format": "CSV"
 ⋮    }
 ⋮  }

Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:
> y
 ⋮
 ● Completed in 84.41s

> Excellent! I've successfully created an S3 table from your CSV file. Here's what was accomplished:
• Table name: observations in the default namespace
• Rows imported: 1,480,409 rows from observations.csv
• Table ID: <tableId>
• Location: hello-world table bucket in the default namespace

The table was automatically created with a schema inferred from your CSV file structure. The data is now stored as an Apache Iceberg table and ready for querying.

Would you like me to:
1. Show you the table schema/structure?
2. Run a sample query to see the data?
3. List all tables in your namespace to confirm the creation?
```

- The LLM inferred from the user's prompt and then executed a Tool called `import_csv_to_table`.
- The `import_csv_to_table` Tool creates a Table from the `observations.csv` file in S3 as per the user's input.
- Verify the Table creation in the S3 Console.

### Checking Table Schema

Execute the following prompt:

```text
Show me the table schema/structure?
```

It responds as follows:

```text
> I'll show you the table schema/structure by querying the observations table to see its columns and data types.

🛠️  Using tool: query_database from mcp server awslabss_3_tables_mcp_server
 ⋮
 ● Running query_database with the param:
 ⋮  {
 ⋮    "query": "DESCRIBE observations",
 ⋮    "options": {
 ⋮      "bucket_name": "hello-world",
 ⋮      "namespace_name": "default",
 ⋮      "region": "<region>"
 ⋮    }
 ⋮  }

Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:
> y
 ⋮
 ● Completed in 2.47s

> Perfect! Here's the schema structure of your observations table:

## Table Schema
| Column Name | Data Type | Description |
|-------------|-----------|-------------|
| DATE | Timestamp(Microseconds, UTC) | Date/time of the observation |
| PATIENT | String (Utf8) | Patient identifier |
| ENCOUNTER | String (Utf8) | Encounter identifier |
| CODE | String (Utf8) | Observation code |
| DESCRIPTION | String (Utf8) | Description of the observation |
| VALUE | String (Utf8) | Observation value |
| UNITS | String (Utf8) | Units of measurement |
| TYPE | String (Utf8) | Type of observation |

The table contains 1,480,409 rows of medical observation data with 8 columns. The schema was automatically inferred from your CSV file, with the DATE column properly recognized as a timestamp with UTC timezone,
and all other columns as string types.

Would you like me to show you a sample of the actual data to see what the records look like?
```

### Querying Table Bucket Metadata

Execute the following prompt:

```text
Query all available metadata about hello-world
```

Enter 'y' when asked to run the MCP Server.

It repeatedly calls the S3 Tables MCP Server to generate the necessary data.

It produces summary information about the target bucket, namespace, and table.

Depending on your environment, it generally responds as follows:

```text
> I'll query all available metadata about the hello-world S3 Tables bucket. Let me gather comprehensive information step by step.

🛠️ Using tool: list_table_buckets from mcp server awslabss_3_tables_mcp_server
⋮
● Running list_table_buckets with the param:
⋮  {
⋮    "bucket_name": "hello-world",
⋮    "options": {
⋮      "region": "<region>"
⋮    }
⋮  }

Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:
> y
⋮
● Completed in 0.961s

...

> Here's a comprehensive overview of all available metadata for the hello-world S3 Tables bucket:

## S3 Table Bucket Metadata
Basic Information:
• Name: hello-world
• ARN: arn:aws:s3tables:<region>:<accountId>:bucket/hello-world
• Table Bucket ID: <tableBucketId>
• Type: customer (user-managed)
• Account ID: <accountId>
• Creation Date: July 24, 2025 at 06:27:50 UTC
• Region: <region>

## Namespace Information
Default Namespace:
• Name: default
• Namespace ID: <namespaceId>
• Creation Date: July 24, 2025 at 06:28:57 UTC
• Account ID: <accountId>
• Table Bucket ID: <tableBucketId>

## Table Information
Table: observations
• Name: observations
• Type: customer (read-write)
• ARN: arn:aws:s3tables:<region>:<accountId>:bucket/hello-world/table/<tableId>
• Namespace: default
• Creation Date: July 24, 2025 at 06:31:08 UTC
• Last Modified: July 24, 2025 at 06:31:17 UTC

## Storage and Metadata Locations
• Data Location: s3://738984bd-26aa-42da-hdnmawzszzpmwg3dy4djy3rqmhe9ause1b--table-s3
• Metadata Location: s3://738984bd-26aa-42da-hdnmawzszzpmwg3dy4djy3rqmhe9ause1b--table-s3/metadata/00001-253b98a3-5c44-4f1e-9588-c4058ec0122e.metadata.json
• UUID: f87189ba3986cd040c0d

## Table Schema
The observations table contains 8 columns:
1. DATE - Timestamp(Microseconds, UTC)
2. PATIENT - String (Utf8)
3. ENCOUNTER - String (Utf8)
4. CODE - String (Utf8)
5. DESCRIPTION - String (Utf8)
6. VALUE - String (Utf8)
7. UNITS - String (Utf8)
8. TYPE - String (Utf8)

...

This appears to be a healthcare-related dataset containing patient observations with timestamps, codes, descriptions, values, and units. The table is actively maintained with automatic compaction and snapshot
management enabled.
```

### Checking Maintenance Job Status

Run the following prompt:

```text
Give me the maintenance Job status of "observations" table
```

The response will be as follows:

```text
> I'll check the maintenance job status for the observations table in the hello-world bucket.

🛠️  Using tool: get_maintenance_job_status from mcp server awslabss_3_tables_mcp_server
 ⋮
 ● Running get_maintenance_job_status with the param:
 ⋮  {
 ⋮    "name": "get_maintenance_job_status",
 ⋮    "arguments": {
 ⋮      "namespace": "default",
 ⋮      "name": "observations",
 ⋮      "region_name": "<region>",
 ⋮      "table_bucket_arn": "arn:aws:s3tables:<region>:<accountId>:bucket/hello-world"
 ⋮    }
 ⋮  }

Allow this action? Use 't' to trust (always allow) this tool for the session. [y/n/t]:
> y
 ⋮
 ● Completed in 1.208s

> Here's the maintenance job status for the observations table:

## Maintenance Job Status Summary
Table: observations
Table ARN: arn:aws:s3tables:<region>:<accountId>:bucket/hello-world/table/<tableId>

### Maintenance Operations Status
1. Iceberg Compaction
• Status: Successful ✅
• Last Run: July 24, 2025 at 06:36:10 UTC
• Purpose: Combines small files into larger ones for better query performance

2. Iceberg Unreferenced File Removal
• Status: Successful ✅
• Last Run: July 24, 2025 at 06:41:54 UTC
• Purpose: Removes orphaned files to optimize storage usage and reduce costs

3. Iceberg Snapshot Management
• Status: Successful ✅
• Last Run: July 24, 2025 at 06:36:46 UTC
• Purpose: Manages table version histories and controls metadata growth

All maintenance operations have completed successfully and are running as expected. The most recent maintenance activities occurred shortly after the table was created (around 06:31 UTC), which indicates the system
is actively maintaining the table's performance and storage efficiency.
```
