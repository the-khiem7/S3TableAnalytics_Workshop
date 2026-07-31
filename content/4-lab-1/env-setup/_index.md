---
title: "Environment Setup"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>5.2. </b>"
---

## 1. Upload CSV file and Apache Iceberg jar in S3

We will upload the CSV data file, Apache Iceberg Runtime Jar file, and Glow Jar file required for this workshop to the S3 bucket.

Download the workshop CSV file,
- AL_D151_11_20240125_UTF8.csv
- AL_D151_41_20250120_200000_UTF8.csv

These files are public data from South Korea.

- **AL_D151_11_20240125_UTF8.csv:** A data file containing publicly announced land prices by region in Seoul.
- **AL_D151_41_20250120_200000_UTF8.csv:** A data file containing 100,000 publicly announced land prices from Seoul and 100,000 publicly announced land prices from Gyeonggi-do. (Will be used for Merge query practice.)

Download the Apache Iceberg runtime jar file for Amazon S3 Tables, **s3-tables-catalog-for-iceberg-runtime-0.1.5.jar**.

Access the [S3 Console](https://console.aws.amazon.com/s3) page.

Select 'General purpose buckets' from the left menu.

Click on the 'workshop-bucket-xxxxxx' bucket.

Click 'Create folder' and create a folder named 'lib'.

Click 'Create folder' again and create a folder named 'data'.

Click 'Upload', then use the 'Add files' button to upload the downloaded files to their respective paths.

{{% notice info %}}
Upload a Jar file (s3-tables-catalog-for-iceberg-runtime-0.1.5.jar) to the 's3://${bucket}/lib/' path.

Upload two CSV files (AL_D151_11_20240125_UTF8.csv and AL_D151_41_20250120_200000_UTF8.csv) to the 's3://${bucket}/data/' path.
{{% /notice %}}

![Upload data file](/images/workshop/upload-data.webp)

![Upload jar file](/images/workshop/upload-jar.webp)

You have completed uploading the files necessary for the workshop to S3.
