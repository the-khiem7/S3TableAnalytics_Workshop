---
title: "[Optional] SQL Playground"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b>4.6. </b>"
---

## Data Transformation Query

{{% notice info %}}
We will run queries that transform the original data. Transformation is typically processed for the following purposes:

- **Data Quality Improvement:** To cleanse incomplete or inaccurate data, remove duplicates, and handle missing values.
- **Data Structuring:** To structure unstructured data or restructure existing structures to fit the target system.
- **Data Standardization:** To unify the formats, units, and naming conventions of data from various sources into a consistent standard.
- **Data Integration:** To meaningfully combine data from multiple sources to provide an integrated view.
- **Business Rule Application:** To apply business logic and rules to data to transform it into meaningful information.
- **Calculation and Aggregation:** To perform necessary calculations and generate summary information or aggregated data.
- **Data Security Enhancement:** To maintain data security by masking or encrypting sensitive information.
- **Data Optimization:** To optimize data structures for improved analysis and query performance.
- **Data Size Management:** To improve storage space and processing efficiency by selecting only necessary data or compressing it.
{{% /notice %}}

The following query is an example of a Transformation query with the above purposes.

It joins patient information and observation data, and aggregates the observation data for a specific patient into a JSON array.

In addition to this, please try writing and running your own Transformation queries.

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

## Data Discovery Queries

{{% notice info %}}
We have prepared several sample queries for data exploration and analysis. Run the queries and see how data exploration and analysis are performed using Iceberg tables.
{{% /notice %}}

### Count the number of patients

```python
# Patient count
spark.sql("""
SELECT COUNT(DISTINCT id) AS patient_count 
FROM s3table.workshop_namespace.patients 
""").show()
```

### Count patients by gender, race, and marital status

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

### Count patients by allergy type

```python
spark.sql("""
SELECT code, description, COUNT(DISTINCT patient) AS patient_count
FROM s3table.workshop_namespace.allergies
GROUP BY code, description
ORDER BY patient_count DESC
""").show(20, False)
```

### Count patients by vaccination

```python
spark.sql("""
SELECT code, description, COUNT(DISTINCT patient) AS patient_count
FROM s3table.workshop_namespace.immunizations
GROUP BY code, description
ORDER BY patient_count DESC
""").show(20, False)
```

### Count encounters by type (e.g., visit, emergency, inpatient)

```python
spark.sql("""
SELECT code, description, COUNT(*) AS encounter_count
FROM s3table.workshop_namespace.encounters 
GROUP BY code, description
ORDER BY encounter_count DESC
""").show(20, False)
```

### Top 10 diagnosed conditions

```python
spark.sql(f"""
SELECT code, description, COUNT(*) AS cnt
FROM s3table.workshop_namespace.conditions 
GROUP BY code, description
ORDER BY cnt DESC
LIMIT 10
""").show(152, False)
```

### Count conditions per patient and find the most diagnosed condition

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

### Analyze medications prescribed for conditions

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

### Time-series analysis of observation data per patient

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

### Analyze condition distribution by age group and gender

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

### Top 10 final diagnosed conditions for deceased patients

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

### Average lifespan of patients diagnosed with conditions

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

### Average lifespan by patient location

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

### Average weight, pulse, and blood pressure for hypertensive patients

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

### Smoking status of hypertensive patients vs. non-hypertensive patients

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

### Analyze the effectiveness of treatment plans

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

### Analysis of Patient Visits and Treatments by Medical Institution

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

We ran some sample queries myself to explore and analyze the data.

Next, we'll look at data visualization.
