---
title: "Option 1: Amazon Athena"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b>5.3. </b>"
---

## Create Source Table

Access the [Amazon Athena Console](https://console.aws.amazon.com/athena) page.

Enter the following values in the left selection field:

- **Data source:** AwsDataCatalog
- **Catalog:** None
- **Database:** No input value

Create a table using the data uploaded to S3.

```sql
CREATE EXTERNAL TABLE IF NOT EXISTS default.source_table (
    id STRING,	
    legal_dong_code STRING,
    legal_dong_name STRING,
    category_code STRING,
    category_name STRING,
    land_number STRING,
    reference_year STRING,
    refernece_month STRING,
    disclosure_price DOUBLE,
    disclosure_date DATE,
    is_standard STRING,
    base_date DATE,
    sido_code STRING
)
COMMENT 'Source table'
ROW FORMAT DELIMITED FIELDS TERMINATED BY ','
STORED AS TEXTFILE
LOCATION 's3://<bucket>/data/'
TBLPROPERTIES (
    'classification'='csv',
    'serialization.encoding'='utf8',
    "skip.header.line.count"="1"
)
```

Write the table creation query to match the original data, which is in CSV format with ',' as the delimiter.
In the LOCATION, enter the S3 path where you uploaded the data earlier.
Replace `<bucket>` with the S3 bucket name appropriate for each environment.
Example: `s3://workshop-bucket-12341234/data/`

![Create table](/images/workshop/athena-create-table.webp)

Check the data in the created table.

```sql
SELECT * FROM default.source_table
```

```sql
SELECT COUNT(*) FROM default.source_table
-- 897963
```

![Check source table](/images/workshop/athena-check-source-table.webp)

The source table creation using the CSV file in S3 is complete.

## Create in S3 Tables

Enter the following values in the left selection field:

- **Data source:** AwsDataCatalog
- **Catalog:** s3tablescatalog/workshop-table-bucket
- **Database:** workshop_namespace

Create a table in Amazon S3 Tables.

```sql
CREATE TABLE workshop_namespace.individually_disclosed_building_price (
    id STRING,	
    legal_dong_code STRING,
    legal_dong_name STRING,
    category_code STRING,
    category_name STRING,
    land_number STRING,
    reference_year STRING,
    refernece_month STRING,
    disclosure_price DOUBLE,
    disclosure_date DATE,
    is_standard STRING,
    base_date DATE,
    sido_code STRING
) COMMENT 'Target table'
```

The creation of the table in Amazon S3 Tables to receive data from the source table is complete.

## Merge into S3 Tables

Enter the following values in the left selection field:

- **Data source:** AwsDataCatalog
- **Catalog:** s3tablescatalog/workshop-table-bucket
- **Database:** workshop_namespace

Merge data from 'source_table' into the Amazon S3 Tables table.

```sql
MERGE INTO workshop_namespace.individually_disclosed_building_price t
USING "AwsDataCatalog"."default"."source_table" s
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
```

![Merge table](/images/workshop/athena-merge-table.webp)

Check the data in the created table.

```sql
SELECT * FROM workshop_namespace.individually_disclosed_building_price
```

```sql
SELECT COUNT(*) AS cnt 
FROM workshop_namespace.individually_disclosed_building_price
-- 897963
```

![Check target table](/images/workshop/athena-check-target-table.webp)

The table creation and data merge have been completed successfully.
