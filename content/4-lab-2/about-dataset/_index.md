---
title: "About Datasets"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b>4.1. </b>"
---

{{% notice info "Information" %}}
https://registry.opendata.aws/synthea-coherent-data/

In the case of patient clinic data, it is difficult to use it for workshop hands-on purposes due to privacy issues. For example, data like MIMIC-III can only be used for limited purposes after obtaining approval for its use. Therefore, we will utilize the artificially generated patient clinic data provided at the above URL.
{{% /notice %}}

This data is compressed in a ZIP file and is a **synthetic dataset** that includes CSV file data, FHIR format data, DICOM image data, and Genome data.

In this workshop, we will proceed using the CSV file data from these files.

---

The CSV data consists of the following 16 CSV files:

- `allergies.csv`: Patient allergy records (707 entries)
- `careplans.csv`: Care plans provided to patients (14,115 entries)
- `conditions.csv`: Diagnosed diseases or conditions (35,874 entries)
- `devices.csv`: Medical devices implanted or used by patients (552 entries)
- `encounters.csv`: Contacts between patients and healthcare institutions (visits, admissions, etc.) (285,339 entries)
- `imaging_studies.csv`: Radiology or medical imaging examination results (6,262 entries)
- `immunizations.csv`: Patient immunization records (35,500 entries)
- `medications.csv`: Records of medications taken or being taken by patients (371,210 entries)
- `observations.csv`: Medical observation values or vital sign data (1,480,409 entries)
- `organizations.csv`: Healthcare service providers (2,535 entries)
- `patients.csv`: Patient basic information (3,539 entries)
- `payer_transitions.csv`: Patient insurance status change history (16,328 entries)
- `payers.csv`: Insurance provider information (10 entries)
- `procedures.csv`: Medical procedure and surgery records (134,385 entries)
- `providers.csv`: Healthcare service providers (13,920 entries)

The above CSV data are related to each other through id values.

For example, the conditions (diagnosis) data includes the id value from the patients data as a column for reference.

We will proceed with Lab-2 using this **synthetic data**.

{{% notice info "Information" %}}
**What is AWS Open Data Program?**

The AWS Open Data Program (ODP) helps democratize access to high-value public datasets by hosting them in Amazon S3 and making them easily discoverable through the Registry of Open Data on AWS (RODA). These datasets can be used by anyone to build services or perform analysis using AWS tools like Amazon EC2, Amazon Athena, AWS Lambda, and Amazon EMR. By hosting datasets on AWS, users can reduce the cost and complexity of data acquisition and focus on deriving insights. Data in the ODP is stored in public S3 buckets, meaning there are no access restrictions. Anyone can download and use the data directly from these S3 locations.
{{% /notice %}}
