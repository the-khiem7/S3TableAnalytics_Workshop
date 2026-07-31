---
title: "[Optional] Visualization"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b>4.7. </b>"
---

## 1. Quicksight Sign-up

Create an account in Quicksight.

Go to the [Amazon Quicksight](https://quicksight.aws.amazon.com/) page.

Click the 'SIGN UP FOR QUICKSIGHT' button.

Enter the following values and click the 'Finish' button to complete the sign-up:

- **Email for account notification:** \<Workshop Email\>
- **QuickSight region:** Asia Pacific (Seoul) or US East (N. Virginia)
- **QuickSight account name:** \<User ID\> -> Must be a unique value.
- **Allow access and autodiscovery for these resources:** Check the following resources:
  - IAM
  - Amazon S3: Check and then configure as follows before clicking the 'Finish' button
  - Amazon Athena
  - Add Pixel-Perfect Reports: Uncheck

![Check S3](/images/workshop/select_s3.webp)

Once the sign-up is complete, the creation complete page will appear, and click the 'GO TO QUICKSIGHT' button to proceed.

![Create complete](/images/workshop/create_complete.webp)

Review the Welcome page and close it.

Quicksight sign-up is complete.

## 2. Access Control

Use AWS Lake Formation to set access control for the tables in the Amazon S3 Tables.

Configure access so that the visualization tool Quicksight can access the tables in the Amazon S3 Tables.

https://docs.aws.amazon.com/AmazonS3/latest/userguide/s3-tables-integrating-aws.html#grant-permissions-tables

### 2.1. Get User ARN

Click on the account information at the top right to copy the Account ID.

![Get Account Id](/images/workshop/get_account_id.webp)

Launch CloudShell by clicking on the CloudShell link at the bottom left.

![Open CloudShell](/images/workshop/open_cloudshell.webp)

Run the following CLI to get the Quicksight Admin ARN.

```bash
aws quicksight list-users --aws-account-id <account_id> --namespace default --region <region>
```

- `<account_id>`: Enter the Account ID value copied in step 1.
- `<region>`: Enter 'ap-northeast-2' or 'us-east-1'.

Copy the value corresponding to "Arn" in the response JSON. This value is the ARN of the account to grant access in Lake Formation.

![Get User ARN](/images/workshop/cloudshell.png)

You have obtained the Quicksight user ARN.

### 2.2. Grant Data Permissions

Go to the [AWS Lake Formation](https://console.aws.amazon.com/lakeformation/) page.

Click the 'Get started' button.

Navigate to the 'Permissions' > 'Data permissions' menu on the left, and click the 'Grant' button.

![Lake Formation](/images/workshop/lakeformation.webp)

Enter the following values:

**Principals**
- Select SAML users and groups
- Enter SAML ARN: Enter the ARN value of the account copied earlier

**LF-Tags or catalog resources**
- Select Named Data Catalog resources
- Catalogs: Check \<account_id\>:s3tablescatalog/workshop-table-bucket
- Databases: Check workshop_namespace
- Tables: Check All tables

**Table permissions**
- Table permissions: Check Super
- Grantable permissions: Check Super

Click the 'Grant' button.

You have set access permissions for the Quicksight user ARN to the tables in the Amazon S3 Tables.

## 3. Visualization

Use Quicksight to visualize the data in the Amazon S3 Tables.

### 3.1. Add Datasets

Go to the [Amazon Quicksight](https://quicksight.aws.amazon.com/) page.

Select 'Datasets' from the left menu.

Click the 'NEW DATASET' button at the top right.

Select 'Athena'.

Enter a desired name for 'Data source name' and click the 'Create data source' button.

Click the 'Use custom SQL' button.

Delete the 'New custom SQL' text and enter the table name 'patients'.

Paste the following query in the input box below.

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.patients
```

Click the 'Edit/Preview data' button.

On the next screen, click the 'Apply' button to view the data.

After viewing the data, click the 'Close' button.

Click the 'Add data' button at the top right.

![Add dataset](/images/workshop/add-dataset.png)

Select 'Data source' in the Select menu, choose the data source you created, and click the 'Select' button.

Repeat steps 6-13 for the following Custom SQLs:

**Custom SQLs**

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.encounters
```

![Add encounters](/images/workshop/add-dataset-encounters.webp)

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.conditions
```

![Add conditions](/images/workshop/add-dataset-conditions.webp)

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.observations
```

![Add observations](/images/workshop/add-dataset-observations.webp)

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.procedures
```

![Add procedures](/images/workshop/add-dataset-procedures.webp)

```sql
SELECT * FROM "s3tablescatalog/workshop-table-bucket".workshop_namespace.medications
```

![Add medications](/images/workshop/add-dataset-medications.webp)

After adding all the Custom SQLs, you should see a screen like the one below.

![Add datasets](/images/workshop/add-datasets.webp)

Data addition is complete.

### 3.2. Join Configuration

Set up the join relationships between the added datasets.

{{% notice info %}}
- patients - encounters -> Left
- encounters - conditions -> left
- encounters - observations -> left
- encounters - procedures -> left
- encounters - medications -> left
{{% /notice %}}

Set up the join between 'patients' and 'encounters' as shown in the image below, and click 'Apply' to apply the changes.

- patients Join Column = id
- encounters Join Column = patient
- Join type = Left

![Join Configuration](/images/workshop/join-config-1.webp)

Drag the 'conditions' data to join with 'encounters'. Set up the join and click 'Apply' to apply the changes.

- encounters Join Column = id
- conditions Join Column = encounter
- Join type = Left

![Join Configuration](/images/workshop/join-config-2.png)

Drag the 'observations' data to join with 'encounters'. Set up the join and click 'Apply' to apply the changes.

- encounters Join Column = id
- observations Join Column = encounter
- Join type = Left

![Join Configuration](/images/workshop/join-config-3.png)

Drag the 'procedures' data to join with 'encounters'. Set up the join and click 'Apply' to apply the changes.

- encounters Join Column = id
- procedures Join Column = encounter
- Join type = Left

![Join Configuration](/images/workshop/join-config-4.png)

Drag the 'medications' data to join with 'encounters'. Set up the join and click 'Apply' to apply the changes.

- encounters Join Column = id
- medications Join Column = encounter
- Join type = Left

![Join Configuration](/images/workshop/join-config-5.png)

After completing the above steps, wait for the full column list and data preview to appear on the left.

![Join Configuration](/images/workshop/join-config-final.webp)

Once the 'Query mode' at the bottom left and the 'SAVE & PUBLISH' button at the top right are enabled, set them as follows:

- Query mode = SPICE
- Click the 'SAVE & PUBLISH' button.

{{% notice info %}}
**What is SPICE query mode?**

SPICE is Amazon QuickSight's in-memory engine designed for fast data processing and visualization. This engine loads data into QuickSight's in-memory storage, greatly improving query performance. Each user is provided with 10GB of SPICE capacity by default. Additional capacity can be purchased if more than the default allocation is needed. It is billed on a monthly basis per GB. In this workshop, we use SPICE mode for faster visualization.
{{% /notice %}}

After the Publish complete message, click the 'CLOSE' button.

Click on the created dataset to view its details. You can see that the REFRESH status is 'In progress'.

![Dataset Details](/images/workshop/dataset-details.webp)

The REFRESH typically takes 5-10 minutes to complete.

Wait for the REFRESH to complete. After completion, you should see the dataset details as shown below.

![Dataset Details](/images/workshop/dataset-details-2.png)

The join configuration between datasets and SPICE setup are complete.

All preparations for the dataset are now complete.

### 3.3. Visualize Dashboard

Feel free to visualize the data.

Click the 'USE IN ANALYSIS' button at the top right.

On the New sheet screen, make the necessary settings and click the 'CREATE' button.

You can now freely visualize the data as shown in the image below.

![Visualize](/images/workshop/visualization.webp)

After publishing the dashboard, you can export it in various formats:

It provides document file types like PDF, links, and embedded dashboards. The embedded code is provided as shown below, and you can also set access permissions for individual users.

```html
<iframe
    width="960"
    height="720"
    src="https://<region>.quicksight.aws.amazon.com/sn/embed/share/accounts/<account_id>/dashboards/<dashboard_id>?directory_alias=<quicksight_user_name>">
</iframe>
```

---

You have visualized the clinic data loaded into the Amazon S3 Tables.

By simply loading data into the Amazon S3 Tables, you can easily analyze and visualize data using various AWS Analytics services.
