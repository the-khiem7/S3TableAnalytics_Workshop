---
title: "Amazon S3 Tables Hands-on Lab with AWS Analytics Services"
date: 2026-07-31
weight: 0
chapter: true
---

## Workshop Overview

{{% notice info %}}
The diagram below shows an example of a data lake-based analytics system that can be built using Amazon S3 and S3 Tables.
{{% /notice %}}

![Workshop diagram](/images/workshop/workshop-diagram.webp)

As shown in the diagram above, by utilizing Amazon S3 Tables service based on Amazon S3 and various AWS Analytics services that can query it, you can quickly build a stable big data analytics environment at a low cost. Amazon S3 is AWS's object storage service that is highly reliable and allows you to use storage at a very affordable price. Data files uploaded to Amazon S3 can be loaded into Amazon S3 Tables based on Apache Iceberg to build a data lake. And the data loaded into Amazon S3 Tables can be easily integrated with AWS Analytics services, allowing you to safely and easily expand your data analysis environment.

**Through this workshop, the blue box section is covered, which represents the most critical part of building a data lake based on Amazon S3 and S3 Tables.**

{{% notice warning %}}
This workshop is only supported through workshop sessions hosted by AWS events, and is not supported for individual user accounts.
{{% /notice %}}

## Datasets

The data covered in this workshop is primarily used in the public sector and includes three types of data:

**However, performing all data operations would be redundant, and it is recommended to choose one dataset that will be helpful for the workshop's progression.**

**For Lab-1, it's a newly added dataset, and sample analysis queries have been added so you can run them directly.**

- [Lab-1] **(Newly Added)** Synthea Clinic data (CSV format)
  - This is the synthesized patient's clinic data.
  - Sample analysis queries have been added.
- [Lab-2] Publicly announced land price data (CSV format)
  - This is public data on publicly announced land prices by region in Seoul.
- [Lab-3] **(Optional)** genomic variant data (VCF format)
  - This is VCF (Variant Call Format) data publicly shared on S3.
  - It's a file format for storing variants found in genomes.
  - This data can be helpful for **Healthcare & Lifesciences** field.

## Target Audience

- Data professionals interested in Amazon S3 Tables service and Apache Iceberg.

## Duration

- Expected to take approximately 2 hours.
- May vary depending on participants' understanding of Python and Spark.

## Required Skills

- Basic knowledge of Python programming
- Understanding of data processing including Spark (Spark SQL, Dataframe, ETL, etc.)

## Expected Outcome

- Overall understanding of Apache Iceberg and Amazon S3 Tables.
- Comprehensive understanding of data processing methods using Spark based on Apache Iceberg tables.

## Region

- `ap-northeast-2`, `us-east-1`
