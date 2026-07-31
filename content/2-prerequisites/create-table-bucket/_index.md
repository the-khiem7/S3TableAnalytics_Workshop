---
title: "Create Table Bucket"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>3.1. </b>"
---

{{% notice info %}}
Create a Table Bucket in the AWS console.
{{% /notice %}}

1. Access the [S3 Console](https://console.aws.amazon.com/s3/home) page.
2. Select 'Table buckets' from the left menu.
3. Click 'Create table bucket'.
4. Enter an appropriate bucket name in 'Table bucket name'. e.g. workshop-table-bucket
5. Check 'Enable integration' in the 'Integration with AWS analytics services' section.
   - ![Create table bucket](/images/workshop/create_table_bucket.webp)
   - When this is enabled, a Catalog named 's3tablescatalog' is created in AWS Glue Catalog.
   - This Catalog will enable integration with AWS analytics services (Athena, Redshift, EMR, etc.) later.
6. Click the 'Create table bucket' button to create the Table bucket.
   - ![Create table bucket](/images/workshop/create_table_bucket_complete.webp)

{{% notice warning %}}
Table Bucket creation is complete.
{{% /notice %}}
