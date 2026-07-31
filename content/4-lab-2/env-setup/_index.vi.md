---
title: "Cấu hình Môi trường"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>4.2. </b>"
---

## Tải tệp CSV và Apache Iceberg jar lên S3

{{% notice info %}}
Trong workshop này, chúng ta sẽ tải tệp Apache Iceberg Runtime Jar cần thiết cho workshop lên S3 bucket.
{{% /notice %}}

1. Truy cập trang [AWS Console](https://console.aws.amazon.com).

2. Nhấp vào nút 'CloudShell' ở cuối trang console để khởi chạy Cloud Shell.
   - ![Cloud Shell](/images/workshop/open_cloudshell.webp)

3. Sao chép dữ liệu CSV vào S3 bucket của workshop bằng lệnh bên dưới.

```bash
BUCKET=$(aws s3api list-buckets --query "Buckets[0].Name" --output text)

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/allergies.csv' --output allergies.csv
aws s3 cp allergies.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/careplans.csv' --output careplans.csv
aws s3 cp careplans.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/conditions.csv' --output conditions.csv
aws s3 cp conditions.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/devices.csv' --output devices.csv
aws s3 cp devices.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/encounters.csv' --output encounters.csv
aws s3 cp encounters.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/imaging_studies.csv' --output imaging_studies.csv
aws s3 cp imaging_studies.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/immunizations.csv' --output immunizations.csv
aws s3 cp immunizations.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/medications.csv' --output medications.csv
aws s3 cp medications.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/observations.csv' --output observations.csv
aws s3 cp observations.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/organizations.csv' --output organizations.csv
aws s3 cp organizations.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/patients_merged.csv' --output patients_merged.csv
aws s3 cp patients_merged.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/patients.csv' --output patients.csv
aws s3 cp patients.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/payer_transitions.csv' --output payer_transitions.csv
aws s3 cp payer_transitions.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/payers.csv' --output payers.csv
aws s3 cp payers.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/procedures.csv' --output procedures.csv
aws s3 cp procedures.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/providers.csv' --output providers.csv
aws s3 cp providers.csv s3://${BUCKET}/data/coherent/

curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/data/coherent/supplies.csv' --output supplies.csv
aws s3 cp supplies.csv s3://${BUCKET}/data/coherent/
```

4. Sao chép tệp Apache Iceberg runtime jar cho Amazon S3 Tables vào S3 bucket của workshop bằng lệnh bên dưới.

```bash
curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar' --output s3-tables-catalog-for-iceberg-runtime-0.1.5.jar
aws s3 cp s3-tables-catalog-for-iceberg-runtime-0.1.5.jar s3://${BUCKET}/lib/
```

{{% notice info "Thông tin" %}}
- Tải các tệp CSV lên đường dẫn 's3://${bucket}/data/coherent/'.

- Tải tệp Jar (`s3-tables-catalog-for-iceberg-runtime-0.1.5.jar`) lên đường dẫn 's3://${bucket}/lib/'.
{{% /notice %}}

{{% notice warning %}}
Bạn đã hoàn tất việc tải các tệp cần thiết lên S3 cho workshop.
{{% /notice %}}
