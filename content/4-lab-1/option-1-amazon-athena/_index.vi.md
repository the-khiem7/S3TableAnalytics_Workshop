---
title: "Tùy chọn 1: Amazon Athena"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b>5.3. </b>"
---

## Tạo Bảng Nguồn

Truy cập trang [Amazon Athena Console](https://console.aws.amazon.com/athena).

Nhập các giá trị sau vào trường lựa chọn bên trái:

- **Data source:** AwsDataCatalog
- **Catalog:** None
- **Database:** Không nhập giá trị

Tạo bảng sử dụng dữ liệu đã tải lên S3.

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

Viết truy vấn tạo bảng phù hợp với dữ liệu gốc ở định dạng CSV với dấu phân cách ','.
Trong LOCATION, nhập đường dẫn S3 nơi bạn đã tải dữ liệu lên trước đó.
Thay thế `<bucket>` bằng tên S3 bucket phù hợp với từng môi trường.
Ví dụ: `s3://workshop-bucket-12341234/data/`

![Tạo bảng](/images/workshop/athena-create-table.webp)

Kiểm tra dữ liệu trong bảng đã tạo.

```sql
SELECT * FROM default.source_table
```

```sql
SELECT COUNT(*) FROM default.source_table
-- 897963
```

![Kiểm tra bảng nguồn](/images/workshop/athena-check-source-table.webp)

Việc tạo bảng nguồn sử dụng file CSV trong S3 đã hoàn tất.

## Tạo trong S3 Tables

Nhập các giá trị sau vào trường lựa chọn bên trái:

- **Data source:** AwsDataCatalog
- **Catalog:** s3tablescatalog/workshop-table-bucket
- **Database:** workshop_namespace

Tạo bảng trong Amazon S3 Tables.

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

Việc tạo bảng trong Amazon S3 Tables để nhận dữ liệu từ bảng nguồn đã hoàn tất.

## Merge vào S3 Tables

Nhập các giá trị sau vào trường lựa chọn bên trái:

- **Data source:** AwsDataCatalog
- **Catalog:** s3tablescatalog/workshop-table-bucket
- **Database:** workshop_namespace

Merge dữ liệu từ 'source_table' vào bảng Amazon S3 Tables.

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

![Merge bảng](/images/workshop/athena-merge-table.webp)

Kiểm tra dữ liệu trong bảng đã tạo.

```sql
SELECT * FROM workshop_namespace.individually_disclosed_building_price
```

```sql
SELECT COUNT(*) AS cnt 
FROM workshop_namespace.individually_disclosed_building_price
-- 897963
```

![Kiểm tra bảng đích](/images/workshop/athena-check-target-table.webp)

Việc tạo bảng và merge dữ liệu đã hoàn thành thành công.
