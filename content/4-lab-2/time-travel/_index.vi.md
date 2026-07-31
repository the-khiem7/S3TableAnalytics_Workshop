---
title: "Time Travel"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b>4.5. </b>"
---

## Lấy lịch sử bảng (Snapshots)

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

Lịch sử Snapshot của bảng được in ra.

Nếu bạn đã làm theo trình tự workshop,

- Hàng trên cùng là Snapshot khi bảng được tạo/chèn
- Hàng thứ hai là Snapshot khi truy vấn Merge được thực thi
- Hàng thứ ba là Snapshot khi truy vấn Update được thực thi
- Hàng thứ tư là Snapshot khi truy vấn Delete được thực thi.

## Lấy thông tin Snapshot

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

Vị trí tệp metadata và dữ liệu liên quan đến tệp của Snapshot được in ra.

## Truy vấn hàng với Update Snapshot

```python
merged_snapshot_id = "<merged_snapshot_id>"

spark.sql(f"""
SELECT * FROM s3table.workshop_namespace.patients
FOR VERSION AS OF {merged_snapshot_id}
WHERE id IN ('merge_id_1', 'merge_id_2')
""").show(5, False)
```

Đặt giá trị `<merged_snapshot_id>` thành giá trị Snapshot ID trước khi merge. (giá trị snapshot_id từ hàng thứ hai của kết quả truy vấn Get Table History (Snapshots) trước đó)

Bạn có thể thấy giá trị cột birthdate có giá trị trước khi được cập nhật thành '2000-01-01'.

## Truy vấn hàng trước thời điểm xóa

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
SELECT * FROM s3table.workshop_namespace.patients
FOR TIMESTAMP AS OF TIMESTAMP '{deletion_time}'
WHERE id IN ('merge_id_1', 'merge_id_2')
""").show(5, False)
```

Đặt `deletion_time` thành thời điểm trước khi xóa dữ liệu. (giá trị made_current_at từ hàng thứ tư của kết quả truy vấn Get Table History (Snapshots) trước đó)

Bạn có thể thấy các hàng đã bị xóa.

## Rollback xóa

```python
deletion_time = "<yyyy-MM-dd HH:mm:ss>"

spark.sql(f"""
CALL s3table.system.rollback_to_timestamp('workshop_namespace.patients', TIMESTAMP '{deletion_time}')
""")

spark.sql(f"""
SELECT COUNT(*) FROM s3table.workshop_namespace.patients
""").show()
```

Đặt `deletion_time` thành thời điểm trước khi xóa dữ liệu.

Bạn có thể thấy dữ liệu đã bị xóa trước đó đã được rollback.

---

Chúng ta đã cùng tìm hiểu về các truy vấn Time Travel của Apache Iceberg.
