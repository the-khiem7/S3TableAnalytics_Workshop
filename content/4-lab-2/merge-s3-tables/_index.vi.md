---
title: "Merge/Update/Delete S3 Tables"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b>4.4. </b>"
---

## Merge bảng

{{% notice info %}}
Hãy thực hiện truy vấn Merge.
{{% /notice %}}

1. Nhấp nút '+' để thêm một Cell.

2. Chèn mã sau vào Cell đã thêm và Run.

```python
merge_df = spark.read.option("header", True).csv(f"s3://{bucket}/data/coherent/patients_merged.csv")
merge_df = merge_df.toDF(*[col.lower() for col in merge_df.columns])

merge_df.createOrReplaceTempView("t_merged_patients")
```

- Chúng ta đọc tệp `patients_merged.csv` đã được chuẩn bị trước để kiểm tra truy vấn Merge.
- Tệp này chứa 2 bản ghi trùng lặp và 2 bản ghi không trùng lặp từ tệp patients.csv mà chúng ta đã tạo thành bảng trước đó.
- Nói cách khác, có tổng cộng 4 bản ghi dữ liệu, trong đó 2 bản ghi trùng lặp với dữ liệu chúng ta đã đưa vào S3 Tables trước đó.
- `merge_df.createOrReplaceTempView("t_merged_patients")` — Phần này đăng ký DataFrame đọc từ tệp CSV dưới dạng Temp View. Sau khi đăng ký, nó có thể được tham chiếu như một bảng trong cú pháp Spark SQL.

3. Nhấp nút '+' để thêm Cell, sau đó chạy mã sau để kiểm tra dữ liệu cần merge.

```sql
%%sql
SELECT * FROM t_merged_patients
```

Bạn sẽ thấy dữ liệu sau.

![Dữ liệu Merge](/images/workshop/merge_data.webp)

4. Nhấp nút '+' để thêm Cell, sau đó chạy mã sau để kiểm tra số lượng bản ghi trước khi merge.

```sql
%%sql
SELECT COUNT(*) FROM s3table.workshop_namespace.patients    
```

Bạn có thể xác nhận có 3539 bản ghi.

5. Nhấp nút '+' để thêm Cell, chèn mã sau và Run.

```python
spark.sql(f"""
    MERGE INTO s3table.workshop_namespace.patients t
    USING t_merged_patients s
    ON (s.id=t.id)
    WHEN MATCHED THEN
        UPDATE SET 
            id = s.id, birthdate = s.birthdate, deathdate = s.deathdate, ssn = s.ssn,
            drivers = s.drivers, passport = s.passport, prefix = s.prefix,
            first = s.first, last = s.last, suffix = s.suffix, maiden = s.maiden,
            marital = s.marital, race = s.race, ethnicity = s.ethnicity,
            gender = s.gender, birthplace = s.birthplace, address = s.address,
            city = s.city, state = s.state, county = s.county, zip = s.zip, lat = s.lat, lon = s.lon,
            healthcare_expenses = s.healthcare_expenses, healthcare_coverage = s.healthcare_coverage            
    WHEN NOT MATCHED THEN
        INSERT (
            id, birthdate, deathdate, ssn, drivers, passport,
            prefix, first, last, suffix, maiden, marital, race, ethnicity, gender,
            birthplace, address, city, state, county, zip, lat, lon,
            healthcare_expenses, healthcare_coverage
        ) VALUES (
            s.id, s.birthdate, s.deathdate, s.ssn, s.drivers, s.passport, 
            s.prefix, s.first, s.last, s.suffix, s.maiden, s.marital, s.race, s.ethnicity, s.gender, 
            s.birthplace, s.address, s.city, s.state, s.county, s.zip, s.lat, s.lon, 
            s.healthcare_expenses, s.healthcare_coverage 
        )
""")
```

- Chúng ta thực hiện truy vấn Merge bằng `spark.sql()`.
- Xem xét truy vấn Merge:
  - `s3table.workshop_namespace.patients` là bảng Target để merge dữ liệu vào.
  - `t_merged_patients` là bảng Source mà chúng ta đã đăng ký trước đó dưới dạng Temp View.
  - Phần sau mệnh đề `ON` thiết lập điều kiện khớp. Vì cột `id` là khóa duy nhất, chúng ta kiểm tra trùng lặp bằng cột này.
  - `WHEN MATCHED THEN` có nghĩa là đối với dữ liệu khớp, câu lệnh UPDATE bên dưới được thực thi.
  - `WHEN NOT MATCHED THEN` có nghĩa là đối với dữ liệu không khớp, câu lệnh INSERT bên dưới được thực thi.
  - Nói cách khác, đây là câu lệnh cập nhật dữ liệu hiện có và chèn dữ liệu mới.

6. Nhấp nút '+' để thêm Cell, sau đó chạy mã sau để kiểm tra số lượng bản ghi sau khi merge.

```sql
%%sql
SELECT COUNT(*) FROM s3table.workshop_namespace.patients
```

Sau khi merge, bạn có thể xác nhận có 3541 bản ghi.
Như đã giải thích trước đó, dữ liệu sử dụng trong Merge chứa tổng cộng 4 bản ghi, trong đó 2 bản ghi đã được tải vào bảng.
Do đó, số hàng chỉ tăng thêm 2.
Chúng ta đã xác nhận rằng 3539 + 2 là 3541, và Merge đã được thực hiện đúng.

Chúng ta đã merge dữ liệu CSV vào bảng đã tạo trước đó.

## Update / Delete các hàng tạm

{{% notice info %}}
Phần này cập nhật hoặc xóa dữ liệu từ bảng S3 Tables.
{{% /notice %}}

### Update các hàng tạm

{{% notice warning %}}
Vui lòng ghi lại thời gian khi bạn thực thi truy vấn update. Nó sẽ được sử dụng trong bài tập liên quan đến snapshot.
{{% /notice %}}

1. Nhấp nút '+' để thêm Cell, sau đó Run mã sau.

```sql
%%sql
UPDATE s3table.workshop_namespace.patients SET birthdate='2000-01-01' WHERE id='merge_id_1'
```

Thao tác này cập nhật birthdate thành '2000-01-01' cho hàng có cột id là 'merge_id_1'.

2. Để xác nhận dữ liệu đã cập nhật, nhấp nút '+' để thêm Cell, sau đó Run mã sau.

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.patients WHERE id='merge_id_1'
```

Bạn có thể xác nhận giá trị cột birthdate đã được cập nhật thành '2000-01-01'.

Việc cập nhật đã hoàn tất thành công.

### Delete các hàng tạm

{{% notice warning %}}
Vui lòng ghi lại thời gian khi bạn thực thi truy vấn delete. Nó sẽ được sử dụng trong bài tập liên quan đến snapshot.
{{% /notice %}}

1. Nhấp nút '+' để thêm Cell, sau đó Run mã sau.

```sql
%%sql
DELETE FROM s3table.workshop_namespace.patients WHERE id IN ('merge_id_1', 'merge_id_2')
```

Thao tác này xóa dữ liệu merge tạm thời có giá trị cột id là 'merge_id_1' hoặc 'merge_id_2'.

2. Nhấp nút '+' để thêm Cell, sau đó Run mã sau để xác nhận dữ liệu đã được xóa đúng.

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.patients WHERE id IN ('merge_id_1', 'merge_id_2')
```

Chúng ta đã xác nhận dữ liệu đã được xóa đúng.

Chúng ta đã tìm hiểu về các truy vấn Merge, Update và Delete trên bảng Apache Iceberg.
