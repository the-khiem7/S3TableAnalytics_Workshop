---
title: "S3 Tables Maintenance Jobs"
date: 2026-07-31
weight: 7
chapter: false
pre: " <b>7. </b>"
---

## 1. Access CloudShell

Access CloudShell to check information related to S3 Tables maintenance jobs.

Go to the [AWS Console Home](https://console.aws.amazon.com/).

Click the **CloudShell** button in the lower left corner.

![Cloud Shell](/images/workshop/open-cloudshell.webp)

A terminal window where you can run AWS CLI will appear.

You have accessed the CloudShell environment where you can use AWS CLI to manage Amazon S3 Tables Maintenance Jobs.

## 2. Table Management CLI

Let's perform operations necessary for managing S3 Tables maintenance jobs, such as checking job status and configuring maintenance jobs.

{{% notice info %}}
If the workshop progresses quickly, there may not be any job execution history after table creation. In this case, errors may occur when executing CLI commands.
{{% /notice %}}

First, let's set the variables related to S3 Tables.

```bash
table_bucket_arn="<table-bucket-arn>"
namespace="<namespace>"
table="<table>"
```

- **table-bucket-arn**: Enter the Table Bucket ARN value appropriate for each environment
- **namespace**: workshop_namespace
- **table**: individually_disclosed_building_price

### 2.1. Get Configuration

#### 2.1.1. Job Status

Check the status of the table's maintenance jobs.

```bash
aws s3tables get-table-maintenance-job-status \
    --table-bucket-arn $table_bucket_arn \
    --namespace $namespace \
    --name $table
```

![Job status](/images/workshop/get-job-status.png)

| Job | Description |
|-----|-------------|
| icebergCompaction | Job that compresses and stores a single data file to a specified size |
| icebergUnreferencedFileRemoval | Job that deletes files no longer managed by snapshots |
| icebergSnapshotManagement | Job that cleans up expired snapshots |

#### 2.1.2. Job Configurations

Check the configuration values of table maintenance jobs.

```bash
aws s3tables get-table-maintenance-configuration \
    --table-bucket-arn $table_bucket_arn \
    --namespace $namespace \
    --name $table
```

![Job config](/images/workshop/get-job-configuration.png)

| Configuration | Description |
|---------------|-------------|
| icebergCompaction > targetFileSizeMB | Data file size after compression processing |
| icebergSnapshotManagement > minSnapshotsToKeep | Minimum number of snapshots |
| icebergSnapshotManagement > maxSnapshotAgeHours | Maximum snapshot retention time |

### 2.2. Update Configuration

#### 2.2.1. Compaction

Change the Compaction-related settings among the table maintenance job configuration values.

```bash
aws s3tables put-table-maintenance-configuration \
    --table-bucket-arn $table_bucket_arn \
    --type icebergCompaction \
    --namespace $namespace \
    --name $table \
    --value='{"status":"enabled","settings":{"icebergCompaction":{"targetFileSizeMB":64}}}'
```

| Parameter | Description |
|-----------|-------------|
| status | The status of the maintenance configuration |
| targetFileSizeMB | The target file size for the table in MB |

#### 2.2.2. Snapshot

Change the Snapshot-related settings among the table maintenance job configuration values.

```bash
aws s3tables put-table-maintenance-configuration \
   --table-bucket-arn $table_bucket_arn \
   --namespace $namespace \
   --name $table \
   --type icebergSnapshotManagement \
   --value '{"status":"enabled","settings":{"icebergSnapshotManagement":{"minSnapshotsToKeep":10,"maxSnapshotAgeHours":120}}}'
```

| Parameter | Description |
|-----------|-------------|
| status | The status of the maintenance configuration |
| minSnapshotsToKeep | The minimum number of snapshots to keep |
| maxSnapshotAgeHours | The maximum age of a snapshot before it can be expired |

#### 2.2.3. Remove Orphan Files

Change the settings to delete files that are no longer managed in the table snapshot.

```bash
aws s3tables put-table-bucket-maintenance-configuration \
   --table-bucket-arn $table_bucket_arn \
   --type icebergUnreferencedFileRemoval \
   --value '{"status":"enabled","settings":{"icebergUnreferencedFileRemoval":{"unreferencedDays":3,"nonCurrentDays":10}}}'
```

| Parameter | Description |
|-----------|-------------|
| status | The status of the maintenance configuration |
| unreferencedDays | The number of days an object has to be unreferenced before it is marked as non-current |
| nonCurrentDays | The number of days an object has to be non-current before it is deleted |

#### 2.2.4. Check Configuration

Check the changed job configuration values.

```bash
aws s3tables get-table-maintenance-configuration \
    --table-bucket-arn $table_bucket_arn \
    --namespace $namespace \
    --name $table
```

![Updated job config](/images/workshop/get-updated-job-configuration.png)

You can confirm the updated configuration values.
