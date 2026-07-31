---
title: "[Lab-3] Queries using VCF Data"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b>6. </b>"
---

{{% notice info %}}
When running PySpark jobs through an Amazon EMR Serverless Application, various Python libraries can be packaged as dependencies. To do this, you can use basic Python functionality, build a virtual environment, or configure PySpark tasks directly to use Python libraries. This page covers each approach.
{{% /notice %}}

This lab involves performing CRUD operations and queries on S3 Tables using genomic variant data (VCF data).

The content overlaps with [Lab-1] Queries with Public Data, with the only difference being the type of data used.

For participants not in the Healthcare & Life Sciences field, it is recommended to proceed with Lab-1.

**About Datasets:** Describes the VCF data.

**Environment Setup:** Environment Setup

Upload the jar files for using Amazon S3 Tables service within EMR Serverless.

**Option 1:** Use the Glow library with VCF data and Amazon S3 Tables.

- Load VCF data using the Glow library, then ingest the data into S3 Tables and perform CRUD (Create/Read/Update/Delete) operations.
- While various query engines can be used, for VCF data, due to dependencies with the Glow library, This lab must be conducted through EMR Studio. (Athena cannot be used)

**Option 2:** Process VCF data using only Pyspark code without the Glow library, and use it with Amazon S3 Tables.
