---
title: "Time Travel"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b>4.5. </b>"
---

## Get Table History (Snapshots)

```python
spark.sql(f"""
SELECT * FROM s3table.workshop_namespace.patients.history
""").show(10, False)
```

```
+-----------------------+-------------------+-------------------+-------------------+
|made_current_at        |snapshot_id        |parent_id          |is_current_ancestor|
+-----------------------+-------------------+-------------------+-------------------+
|2025-04-02 02:26:45.356|1996392549628528971|NULL               |true               |
|2025-04-02 02:27:16.224|2842948084683911314|1996392549628528971|true               |
|2025-04-02 02:27:37.664|4907549559143542142|2842948084683911314|true               |
|2025-04-02 02:28:08.795|5291307565494698844|4907549559143542142|true               |
+-----------------------+-------------------+-------------------+-------------------+
```

The Snapshot history of the table is printed.

If you followed the workshop sequence,

- The top row is the Snapshot when the table was created/inserted
- The second is the Snapshot when the Merge query was executed
- The third is the Snapshot when the Update query was executed
- The fourth is the Snapshot when the Delete query was executed.

## Get Snapshot Information

```python
snapshot_id = "<snapshot_id>"

spark.sql(f"""
SELECT * FROM s3table.workshop_namespace.patients.snapshots
WHERE snapshot_id = {snapshot_id} 
""").show(5, False)
```

```
|2025-04-02 02:26:45.356|1996392549628528971|NULL     |append   |s3://00efe3a0-b35c-4718-6rpzzrwd44nayru6un6tp1pqrtbt6apn2b--table-s3/metadata/snap-1996392549628528971-1-69260974-306c-414f-8412-fbadd33fe0bb.avro|{spark.app.id -> 00fre5g0s029592q_0001, added-data-files -> 12, added-records -> 897963, added-files-size -> 5431053, changed-partition-count -> 1, total-records -> 897963, total-files-size -> 5431053, total-data-files -> 12, total-delete-files -> 0, total-position-deletes -> 0, total-equality-deletes -> 0, engine-version -> 3.5.4-amzn-0, app-id -> 00fre5g0s029592q_0001, engine-name -> spark, iceberg-version -> Apache Iceberg unknown (commit unknown)}|
```

The metadata file location and file-related data of the Snapshot are printed.

## Select Rows with Update Snapshot

```python
merged_snapshot_id = "<merged_snapshot_id>"

spark.sql(f"""
SELECT * FROM s3table.workshop_namespace.patients
FOR VERSION AS OF {merged_snapshot_id}
WHERE id IN ('merge_id_1', 'merge_id_2')
""").show(5, False)
```

Set the `<merged_snapshot_id>` value to the Snapshot ID value before the merge. (snapshot_id value from the second row of the previous Get Table History (Snapshots) query results)

You can see that the birthdate column value has the value before it was updated to '2000-01-01'.

## Select Rows before Deletion Time

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
SELECT * FROM s3table.workshop_namespace.patients
FOR TIMESTAMP AS OF TIMESTAMP '{deletion_time}'
WHERE id IN ('merge_id_1', 'merge_id_2')
""").show(5, False)
```

Set `deletion_time` to the time before data deletion. (made_current_at value from the fourth row of the previous Get Table History (Snapshots) query results))

You can see the rows that were deleted.

## Deletion Rollback

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
CALL s3table.system.rollback_to_timestamp('workshop_namespace.patients', TIMESTAMP '{deletion_time}')
""")

spark.sql(f"""
SELECT COUNT(*) FROM s3table.workshop_namespace.patients
""").show()
```

Set `deletion_time` to the time before data deletion.

You can see that the previously deleted data has been rolled back.

---

We have looked at Apache Iceberg's Time Travel queries together.
