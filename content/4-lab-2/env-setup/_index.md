---
title: "Environment Configuration"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b>4.2. </b>"
---

## Upload CSV file and Apache Iceberg jar in S3

{{% notice info %}}
In this workshop, we will upload Apache Iceberg Runtime Jar file required for the workshop to the S3 bucket.
{{% /notice %}}

1. Access the [AWS Console](https://console.aws.amazon.com) page.

2. Click on 'CloudShell' button at the bottom of the console page to launch Cloud Shell.
   - ![Cloud Shell](/images/workshop/open_cloudshell.webp)

3. Copy the CSV data to the workshop S3 bucket using the command below.

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

4. Copy the Apache Iceberg runtime jar file for Amazon S3 Tables to the workshop S3 bucket using the command below.

```bash
curl 'https://static.us-east-1.prod.workshops.aws/public/108ebe5f-ea98-4c98-aece-93c5da46b262/assets/lib/s3-tables-catalog-for-iceberg-runtime-0.1.5.jar' --output s3-tables-catalog-for-iceberg-runtime-0.1.5.jar
aws s3 cp s3-tables-catalog-for-iceberg-runtime-0.1.5.jar s3://${BUCKET}/lib/
```

{{% notice info "Information" %}}
- Upload CSV files to the 's3://${bucket}/data/coherent/' path.

- Upload the Jar file (`s3-tables-catalog-for-iceberg-runtime-0.1.5.jar`) to the 's3://${bucket}/lib/' path.
{{% /notice %}}

{{% notice warning %}}
You have completed uploading the necessary files to S3 for the workshop.
{{% /notice %}}
