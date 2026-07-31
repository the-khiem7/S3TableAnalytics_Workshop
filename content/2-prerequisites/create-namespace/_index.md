---
title: "Create Namespace"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>3.2. </b>"
---

## S3 setting for Athena query result

{{% notice info %}}
Specify the S3 location to store Athena query results. If not specified, queries cannot be executed.
{{% /notice %}}

1. Access the [Amazon Athena Console](https://console.aws.amazon.com/athena/home) page.

2. Click the 'Explore the query editor' button on the right.

3. Click the 'Edit settings' button to specify the S3 location for storing Athena query results. (If not specified, Athena queries cannot be executed.)
   - ![Query result S3 setting](/images/workshop/athena_query_result_s3_setting.webp)

   1. Click the 'Browse S3' button.
   2. Select an existing S3 bucket and click the 'Choose' button.
   3. Confirm that the selected bucket is entered in the 'Location of query result'.
      - You can add a sub-path after the bucket name.
      - Example: s3://workshop-bucket-2ybfnmiyhg8a/query_result
   4. Click the 'Save' button to complete the setting.
   5. After setting, click the 'Editor' tab to move to the query editor page.

{{% notice warning %}}
The S3 storage location setting for Athena query results has been completed.
{{% /notice %}}

---

## Create Namespace

{{% notice info %}}
Create a Namespace to be used in S3 Tables.
{{% /notice %}}

1. Set the left settings on the Editor page as follows:
   - ![Create namespace](/images/workshop/athena_create_namespace.webp)
   - Data source: `AwsDataCatalog`
   - Catalog: `s3tablescatalog/workshop-table-bucket`

2. Enter the following Namespace creation query in the Query Editor:

```sql
CREATE DATABASE IF NOT EXISTS workshop_namespace
```

3. Click the 'Run' button to execute the above query.

4. If the query is executed successfully, you will see the created 'workshop_namespace' in the 'Database' section of the left settings pane.

{{% notice warning %}}
Namespace creation has been completed.
{{% /notice %}}
