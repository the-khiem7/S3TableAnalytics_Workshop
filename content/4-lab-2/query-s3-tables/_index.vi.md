---
title: "[Tùy chọn] SQL Playground"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b>4.6. </b>"
---

## Truy vấn chuyển đổi dữ liệu

{{% notice info %}}
Chúng ta sẽ chạy các truy vấn chuyển đổi dữ liệu gốc. Chuyển đổi thường được xử lý cho các mục đích sau:

- **Cải thiện chất lượng dữ liệu:** Để làm sạch dữ liệu không đầy đủ hoặc không chính xác, loại bỏ trùng lặp và xử lý giá trị thiếu.
- **Cấu trúc hóa dữ liệu:** Để cấu trúc dữ liệu phi cấu trúc hoặc tái cấu trúc các cấu trúc hiện có cho phù hợp với hệ thống đích.
- **Chuẩn hóa dữ liệu:** Để thống nhất định dạng, đơn vị và quy ước đặt tên của dữ liệu từ các nguồn khác nhau thành một tiêu chuẩn nhất quán.
- **Tích hợp dữ liệu:** Để kết hợp có ý nghĩa dữ liệu từ nhiều nguồn nhằm cung cấp cái nhìn tổng hợp.
- **Áp dụng quy tắc nghiệp vụ:** Để áp dụng logic và quy tắc nghiệp vụ vào dữ liệu nhằm chuyển đổi thành thông tin có ý nghĩa.
- **Tính toán và tổng hợp:** Để thực hiện các phép tính cần thiết và tạo thông tin tóm tắt hoặc dữ liệu tổng hợp.
- **Tăng cường bảo mật dữ liệu:** Để duy trì bảo mật dữ liệu bằng cách che giấu hoặc mã hóa thông tin nhạy cảm.
- **Tối ưu hóa dữ liệu:** Để tối ưu cấu trúc dữ liệu nhằm cải thiện hiệu suất phân tích và truy vấn.
- **Quản lý kích thước dữ liệu:** Để cải thiện không gian lưu trữ và hiệu quả xử lý bằng cách chỉ chọn dữ liệu cần thiết hoặc nén nó.
{{% /notice %}}

Truy vấn sau là ví dụ về truy vấn Transformation với các mục đích trên.

Nó join thông tin bệnh nhân và dữ liệu quan sát, đồng thời tổng hợp dữ liệu quan sát cho một bệnh nhân cụ thể thành mảng JSON.

Ngoài ra, hãy thử viết và chạy các truy vấn Transformation của riêng bạn.

```python
spark.sql(f"""
    SELECT
        id, birthdate, deathdate, ssn, drivers, passport, prefix, first, last, suffix, maiden, marital, race, ethnicity, gender, 
        birthplace, address, city, state, county, zip, lat, lon, healthcare_expenses, healthcare_coverage, 
        TO_JSON(ARRAY_AGG(NAMED_STRUCT("date", date, "encounter", encounter, "code", code, "description", description, 
                                       "value", value, "units", units, "type", type))) AS observations
    FROM (
        SELECT a.*, b.*
        FROM s3table.workshop_namespace.observations a 
        LEFT OUTER JOIN s3table.workshop_namespace.patients b
        ON a.patient = b.id
    )
    WHERE id='0020dcbe-9f8d-920d-c008-d68debcef322'
    GROUP BY id, birthdate, deathdate, ssn, drivers, passport, 
        prefix, first, last, suffix, maiden, marital, race, ethnicity, gender, 
        birthplace, address, city, state, county, zip, lat, lon, 
        healthcare_expenses, healthcare_coverage
""").show(30, False, vertical=True)
```

## Truy vấn khám phá dữ liệu

{{% notice info %}}
Chúng tôi đã chuẩn bị một số truy vấn mẫu để khám phá và phân tích dữ liệu. Chạy các truy vấn và xem cách khám phá và phân tích dữ liệu được thực hiện bằng bảng Iceberg.
{{% /notice %}}

### Đếm số lượng bệnh nhân

```python
# Patient count
spark.sql("""
SELECT COUNT(DISTINCT id) AS patient_count 
FROM s3table.workshop_namespace.patients 
""").show()
```

### Đếm bệnh nhân theo giới tính, chủng tộc và tình trạng hôn nhân

```python
spark.sql("""
SELECT gender, COUNT(*) AS patient_count
FROM s3table.workshop_namespace.patients
GROUP BY gender
""").show()

spark.sql("""
SELECT race, COUNT(*) AS patient_count
FROM s3table.workshop_namespace.patients
GROUP BY race
""").show()

spark.sql("""
SELECT marital, COUNT(*) AS patient_count
FROM s3table.workshop_namespace.patients
GROUP BY marital
""").show()
```

### Đếm bệnh nhân theo loại dị ứng

```python
spark.sql("""
SELECT code, description, COUNT(DISTINCT patient) AS patient_count
FROM s3table.workshop_namespace.allergies
GROUP BY code, description
ORDER BY patient_count DESC
""").show(20, False)
```

### Đếm bệnh nhân theo loại tiêm chủng

```python
spark.sql("""
SELECT code, description, COUNT(DISTINCT patient) AS patient_count
FROM s3table.workshop_namespace.immunizations
GROUP BY code, description
ORDER BY patient_count DESC
""").show(20, False)
```

### Đếm lượt khám theo loại (ví dụ: khám, cấp cứu, nội trú)

```python
spark.sql("""
SELECT code, description, COUNT(*) AS encounter_count
FROM s3table.workshop_namespace.encounters 
GROUP BY code, description
ORDER BY encounter_count DESC
""").show(20, False)
```

### Top 10 tình trạng được chẩn đoán nhiều nhất

```python
spark.sql(f"""
SELECT code, description, COUNT(*) AS cnt
FROM s3table.workshop_namespace.conditions 
GROUP BY code, description
ORDER BY cnt DESC
LIMIT 10
""").show(152, False)
```

### Đếm tình trạng trên mỗi bệnh nhân và tìm tình trạng được chẩn đoán nhiều nhất

```python
spark.sql("""
WITH patient_conditions AS (
    SELECT
        p.id,
        p.first,
        p.last,
        COUNT(c.code) AS condition_count,
        ARRAY_AGG(STRUCT(c.code, c.description)) AS conditions
    FROM s3table.workshop_namespace.patients p
    JOIN s3table.workshop_namespace.conditions c ON p.id = c.patient
    GROUP BY p.id, p.first, p.last
),
ranked_conditions AS (
    SELECT
        id,
        first,
        last,
        condition_count,
        conditions,
        ROW_NUMBER() OVER (ORDER BY condition_count DESC) AS rank
    FROM patient_conditions
)
SELECT
    id,
    first,
    last,
    condition_count,
    conditions
FROM ranked_conditions
WHERE rank <= 10
ORDER BY condition_count DESC
""").show(10, False)
```

### Phân tích thuốc được kê đơn cho các tình trạng

```python
spark.sql("""
WITH condition_medications AS (
    SELECT
        c.code AS condition_code,
        c.description AS condition_name,
        m.code AS medication_code,
        m.description AS medication_name,
        COUNT(DISTINCT c.patient) AS patient_count
    FROM s3table.workshop_namespace.conditions c
    JOIN s3table.workshop_namespace.encounters e ON c.encounter = e.id
    JOIN s3table.workshop_namespace.medications m ON e.id = m.encounter
    GROUP BY c.code, c.description, m.code, m.description
),
ranked_medications AS (
    SELECT
        condition_code,
        condition_name,
        medication_name,
        patient_count,
        ROW_NUMBER() OVER (PARTITION BY condition_code ORDER BY patient_count DESC) AS rank
    FROM condition_medications
),
top_medications AS (
    SELECT
        condition_code,
        condition_name,
        COLLECT_LIST(STRUCT(medication_name, patient_count)) AS top_medications,
        COUNT(*) OVER (PARTITION BY condition_code) AS medication_count
    FROM ranked_medications
    WHERE rank <= 5
    GROUP BY condition_code, condition_name
)
SELECT
    condition_code,
    condition_name,
    top_medications
FROM top_medications
ORDER BY medication_count DESC
LIMIT 20
""").show(20, False)
```

### Phân tích chuỗi thời gian dữ liệu quan sát theo bệnh nhân

```python
spark.sql("""
WITH vital_signs AS (
    SELECT
        o.patient,
        o.date,
        o.code,
        o.description,
        o.value,
        o.units,
        ROW_NUMBER() OVER (PARTITION BY o.patient, o.code ORDER BY o.date) AS reading_number
    FROM s3table.workshop_namespace.observations o
    WHERE o.code IN ('8462-4', '8480-6', '8867-4', '9279-1')  -- Codes for blood pressure, pulse, respiration, temperature, etc.
),
measurement_stats AS (
    SELECT
        p.id,
        p.first,
        p.last,
        vs.code,
        vs.description,
        MIN(vs.value) AS min_value,
        MAX(vs.value) AS max_value,
        AVG(vs.value) AS avg_value,
        STDDEV(vs.value) AS stddev_value,
        COUNT(*) AS measurement_count
    FROM s3table.workshop_namespace.patients p
    JOIN vital_signs vs ON p.id = vs.patient
    GROUP BY p.id, p.first, p.last, vs.code, vs.description
    HAVING COUNT(*) >= 5  -- Only cases with at least 5 measurement values
)
SELECT
    ms.id,
    ms.first,
    ms.last,
    ms.code,
    ms.description,
    vs.date,
    vs.value,
    vs.units,
    ms.min_value,
    ms.max_value,
    ms.avg_value,
    ms.stddev_value
FROM measurement_stats ms
JOIN vital_signs vs ON ms.id = vs.patient AND ms.code = vs.code
ORDER BY ms.id, ms.code, vs.date
""").show(100, False)
```

### Phân tích phân bố tình trạng theo nhóm tuổi và giới tính

```python
spark.sql("""
WITH patient_age_groups AS (
    SELECT
        p.id,
        p.gender,
        FLOOR(DATEDIFF(CURRENT_DATE(), p.birthdate) / 365.25 / 10) * 10 AS age_group
    FROM s3table.workshop_namespace.patients p
)
SELECT
    CONCAT(age_group, '-', age_group + 9) AS age_range,
    gender,
    c.code,
    c.description,
    COUNT(DISTINCT pag.id) AS patient_count
FROM patient_age_groups pag
JOIN s3table.workshop_namespace.conditions c ON pag.id = c.patient
GROUP BY age_group, gender, c.code, c.description
HAVING COUNT(DISTINCT pag.id) >= 10
ORDER BY age_group, gender, patient_count DESC
""").show(100, False)
```

### Top 10 tình trạng chẩn đoán cuối cùng của bệnh nhân đã mất

```python
spark.sql(f"""
WITH patient_conditions AS (
    SELECT
        p.id,
        c.code,
        c.description,
        DATE_DIFF(p.deathdate, c.start) AS days_before_death,
        ROW_NUMBER() OVER (PARTITION BY p.id ORDER BY DATE_DIFF(p.deathdate, c.start) ASC) AS rank
    FROM s3table.workshop_namespace.patients p
    JOIN s3table.workshop_namespace.conditions c 
        ON p.id = c.patient
    WHERE p.deathdate IS NOT NULL
)
SELECT
  code,
  description,
  COUNT(*) AS cnt
FROM patient_conditions
WHERE rank <= 3 AND days_before_death <= 30
GROUP BY code, description
ORDER BY cnt DESC
LIMIT 10
""").show(100, False)
```

### Tuổi thọ trung bình của bệnh nhân được chẩn đoán tình trạng

```python
spark.sql("""
WITH patient_conditions AS (
    SELECT
        c.code,
        c.description,
        DATE_DIFF(p.deathdate, p.birthdate)/365 AS life_span,
        DATE_DIFF(p.deathdate, c.start) AS days_before_death,
        p.id,
        ROW_NUMBER() OVER (PARTITION BY p.id ORDER BY DATE_DIFF(p.deathdate, c.start) ASC) AS rank
    FROM s3table.workshop_namespace.patients p
    JOIN s3table.workshop_namespace.conditions c 
        ON p.id = c.patient
    WHERE p.deathdate IS NOT NULL
)
SELECT
    code, description, AVG(life_span) AS avg_life_span
FROM patient_conditions
WHERE rank <= 3 AND days_before_death <= 30
GROUP BY code, description
HAVING COUNT(DISTINCT id) >= 10
ORDER BY avg_life_span ASC
""").show(150, False)
```

### Tuổi thọ trung bình theo vị trí bệnh nhân

```python
spark.sql("""
WITH patients AS (
    SELECT
        id,
        DATE_DIFF(IF(deathdate IS NULL, CURRENT_DATE(), deathdate), birthdate)/365 AS life_span,
        state,
        city,
        county,
        zip
    FROM s3table.workshop_namespace.patients
    WHERE deathdate IS NOT NULL
)
SELECT
    county, AVG(life_span) AS avg_life_span
FROM patients
GROUP BY county
ORDER BY avg_life_span DESC
LIMIT 100
""").show(100, False)
```

### Cân nặng, mạch và huyết áp trung bình của bệnh nhân tăng huyết áp

```python
# conditions.code='59621000' -> Hypertension diagnosis
# observations.code='8462-4' -> Diastolic blood pressure measurement
# observations.code='8480-6' -> Systolic blood pressure measurement
# observations.code='29463-7' -> Body weight measurement
# observations.code='39156-5' -> Body mass index
spark.sql("""
WITH hightension_avg AS (
    SELECT code, description, AVG(value) AS val, units 
    FROM s3table.workshop_namespace.observations
    WHERE patient IN (SELECT patient FROM s3table.workshop_namespace.conditions WHERE code='59621000')
        AND code IN ('8462-4', '8480-6', '29463-7', '39156-5')
    GROUP BY code, description, units
),
none_hightension_avg AS (
    SELECT code, description, AVG(value) AS val, units 
    FROM s3table.workshop_namespace.observations
    WHERE patient NOT IN (SELECT patient FROM s3table.workshop_namespace.conditions WHERE code='59621000')
        AND code IN ('8462-4', '8480-6', '29463-7', '39156-5')
    GROUP BY code, description, units
)
SELECT
    a.code, a.description, b.val AS hightension_avg, a.val AS none_hightension_avg, a.units
FROM hightension_avg a 
LEFT JOIN none_hightension_avg b
    ON a.code=b.code AND a.description=b.description
ORDER BY description
""").show(20, False)
```

### Tình trạng hút thuốc của bệnh nhân tăng huyết áp so với bệnh nhân không tăng huyết áp

```python
# b.code='59621000' -> Hypertension diagnosis
# a.code='72166-2' -> Smoking measurement
spark.sql("""
WITH hightension_smoking AS (
    SELECT DISTINCT patient, value, 'hightension' AS status
    FROM s3table.workshop_namespace.observations
    WHERE patient IN (SELECT patient FROM s3table.workshop_namespace.conditions WHERE code='59621000')
        AND code='72166-2'
),
none_hightension_smoking AS (
    SELECT DISTINCT patient, value, 'none_hightension' AS status
    FROM s3table.workshop_namespace.observations
    WHERE patient NOT IN (SELECT patient FROM s3table.workshop_namespace.conditions WHERE code='59621000')
        AND code='72166-2'
),
combined_data AS (
    SELECT * FROM hightension_smoking
    UNION
    SELECT * FROM none_hightension_smoking
),
counts AS (
    SELECT
        status,
        value,
        COUNT(*) AS cnt,
        SUM(COUNT(*)) OVER (PARTITION BY status) AS total_per_status
    FROM combined_data
    GROUP BY status, value
)
SELECT
    CONCAT(status, '-', value) AS category,
    cnt,
    total_per_status,
    CAST(cnt AS DOUBLE) / total_per_status AS rate
FROM counts
ORDER BY category
""").show(100, False, vertical=True)
```

### Phân tích hiệu quả của kế hoạch điều trị

```python
spark.sql("""
WITH treatment_outcomes AS (
    SELECT
        cp.patient,
        cp.description AS careplan_description,
        cp.start AS careplan_start,
        cp.stop AS careplan_stop,
        c.code AS condition_code,
        c.description AS condition_description,
        c.start AS condition_start,
        c.stop AS condition_stop,
        CASE
            WHEN c.stop IS NOT NULL AND cp.stop IS NOT NULL THEN 'Resolved'
            WHEN c.stop IS NULL AND cp.stop IS NOT NULL THEN 'Plan Completed, Condition Ongoing'
            WHEN c.stop IS NULL AND cp.stop IS NULL THEN 'Ongoing Treatment'
            ELSE 'Other'
        END AS outcome_status
    FROM s3table.workshop_namespace.careplans cp
    JOIN s3table.workshop_namespace.conditions c
        ON cp.patient = c.patient
        AND c.start <= cp.start
)
SELECT
    condition_code,
    condition_description,
    careplan_description,
    outcome_status,
    COUNT(*) AS count,
    AVG(DATEDIFF(COALESCE(careplan_stop, CURRENT_DATE()), careplan_start)) AS avg_treatment_days
FROM treatment_outcomes
GROUP BY condition_code, condition_description, careplan_description, outcome_status
HAVING COUNT(*) >= 5
ORDER BY condition_code, count DESC
""").show(100, False)
```

### Phân tích lượt khám và điều trị của bệnh nhân theo cơ sở y tế

```python
spark.sql("""
WITH org_encounters AS (
    SELECT
        org.id AS org_id,
        org.name AS organization_name,
        COUNT(DISTINCT e.patient) AS unique_patients,
        COUNT(e.id) AS total_encounters,
        AVG(DATEDIFF(e.stop, e.start)) AS avg_encounter_duration_days,
        COUNT(DISTINCT p.encounter) AS unique_procedures
    FROM s3table.workshop_namespace.organizations org
    JOIN s3table.workshop_namespace.encounters e ON org.id = e.organization
    LEFT JOIN s3table.workshop_namespace.procedures p ON e.id = p.encounter
    GROUP BY org.id, org.name
),
procedure_ranks AS (
    SELECT
        org.id AS org_id,
        p.description AS procedure_description,
        ROW_NUMBER() OVER (PARTITION BY org.id ORDER BY COUNT(*) DESC) AS rank
    FROM s3table.workshop_namespace.organizations org
    JOIN s3table.workshop_namespace.encounters e ON org.id = e.organization
    LEFT JOIN s3table.workshop_namespace.procedures p ON e.id = p.encounter
    WHERE p.description IS NOT NULL
    GROUP BY org.id, p.description
),
top_procedures AS (
    SELECT
        org_id,
        COLLECT_LIST(procedure_description) AS common_procedures
    FROM procedure_ranks
    WHERE rank <= 10
    GROUP BY org_id
)
SELECT
    oe.organization_name,
    oe.unique_patients,
    oe.total_encounters,
    oe.avg_encounter_duration_days,
    oe.unique_procedures,
    tp.common_procedures
FROM org_encounters oe
LEFT JOIN top_procedures tp ON oe.org_id = tp.org_id
ORDER BY oe.unique_patients DESC
""").show(100, False)
```

---

Chúng ta đã tự chạy một số truy vấn mẫu để khám phá và phân tích dữ liệu.

Tiếp theo, chúng ta sẽ tìm hiểu về trực quan hóa dữ liệu.
