---
title: "[Optional] Visualization"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b>5.5. </b>"
---

## 1. Quicksight Sign-up

Sign up for a Quicksight account.

Go to the [Amazon Quicksight](https://quicksight.aws.amazon.com/) page.

Click the 'SIGN UP FOR QUICKSIGHT' button.

Enter the following values and click the 'Finish' button to complete the registration:

- **Email for account notification:** \<Workshop Email\>
- **QuickSight region:** Asia Pacific (Seoul) or US East (N. Virginia)
- **QuickSight account name:** \<User ID\> -> This must be a unique value.
- **Allow access and autodiscovery for these resources:** \<Check the resources below\>
  - IAM
  - Amazon S3: Check and set as follows, then click the 'Finish' button
  - Amazon Athena
  - Add Pixel-Perfect Reports: Uncheck

![Check S3](/images/workshop/quicksight-select-s3.webp)

Once the registration is complete, a creation completion page will appear. Click the 'GO TO QUICKSIGHT' button to proceed.

![Create complete](/images/workshop/quicksight-create-complete.webp)

Review the Welcome page and close it.

Quicksight registration is now complete.

## 2. Access Control

Use AWS Lake Formation to set up permission controls for tables in Amazon S3 Tables.

We will configure settings to allow the visualization tool Quicksight to access tables in Amazon S3 Tables.

This step is crucial for enabling Quicksight to visualize data from Amazon S3 Tables.

### 2.1. Get User ARN

Click on the account information in the upper right corner to copy the Account ID.

![Get Account Id](/images/workshop/quicksight-get-account-id.webp)

Launch CloudShell by clicking the CloudShell link at the bottom left.

![Open CloudShell](/images/workshop/quicksight-open-cloudshell.webp)

Run the following CLI command to check the Quicksight Admin ARN:

```bash
aws quicksight list-users --aws-account-id <account_id> --namespace default --region <region>
```

- `account_id`: Enter the Account ID value you copied in step 1.
- `region`: Enter 'ap-northeast-2' or 'us-east-1'.
- Copy the value corresponding to the "Arn" field from the response JSON.
- This value is the ARN of the Account that will be given access permissions in Lake Formation.

![Get User ARN](/images/workshop/quicksight-cloudshell.webp)

You have obtained the Quicksight user ARN value.

### 2.2. Grant Data Permissions

Go to the [AWS Lake Formation](https://console.aws.amazon.com/lakeformation) page.

Click the 'Get started' button.

Navigate to 'Permissions' > 'Data permissions' in the left menu, then click the 'Grant' button.

Enter the following values:

**Principals**
- Select SAML users and groups
- Enter SAML ARN: Input the Account's ARN value you copied earlier

**LF-Tags or catalog resources**
- Select Named Data Catalog resources
- Catalogs: Check \<account_id\>:s3tablescatalog/workshop-table-bucket
- Databases: Check workshop_namespace
- Tables: Check All tables

**Table permissions**
- Table permissions: Check Super
- Grantable permissions: Check Super

![Lake Formation](/images/workshop/lakeformation.webp)

Click the 'Grant' button.

You have set up access permissions for the Quicksight user ARN to tables in Amazon S3 Tables.

## 3. Visualization

Use Quicksight to visualize data from Amazon S3 Tables.

### 3.1. Datasets

Go to the [Amazon Quicksight](https://quicksight.aws.amazon.com/) page.

Select 'Datasets' from the left menu.

Click the 'New Dataset' button in the upper right corner.

Select 'Athena'.

Enter a desired name for 'Data source name' and click the 'Create data source' button.

Click the 'Use Custom SQL' button.

Enter the following query and click the 'Edit/Preview data' button:

```sql
SELECT * 
FROM "s3tablescatalog"."workshop_namespace"."individually_disclosed_building_price"
```

![Add datasource](/images/workshop/quicksight-add-datasource.webp)

You will be directed to a screen like the one below. Click the 'Apply' button to view the data.

[Note] 'SPICE' is a feature related to data caching that improves query speed.

![Add datasource](/images/workshop/quicksight-add-datasource-2.webp)

Once the data is displayed correctly, click the 'Close' button.

You will be directed to a screen like the one below. Click the 'PUBLISH & VISUALIZE' button.

Click the 'CREATE' button in the New sheet.

![Visualize](/images/workshop/quicksight-visualize.webp)

You have successfully published a table from Amazon S3 Tables as a Dataset in Quicksight.

### 3.2. Visualize Dashboard

Feel free to create your own visualizations.

When you publish a Dashboard, you can export it in various formats:
- It provides document type files like PDF, Links, Embedded dashboards, etc.
- An Embedded code is provided as shown below, and access permissions can be set for each user.

```html
<iframe
    width="600"
    height="400"
    src="https://us-east-1.quicksight.aws.amazon.com/sn/embed/share/accounts/111122223333/dashboards/1a1ac6ed-39c2-4711-8220-99e6e7fa20b0?directory_alias=yourcompany">
</iframe>
```
