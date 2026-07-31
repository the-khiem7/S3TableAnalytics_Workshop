---
title: "Merge/Update/Delete S3 Tables"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b>4.4. </b>"
---

## Merge Table

{{% notice info %}}
Let's perform a Merge query.
{{% /notice %}}

1. Click the '+' button to add a Cell.

2. Insert the following code into the added Cell and Run it.

```python
merge_df = spark.read.option("header", True).csv(f"s3://{bucket}/data/coherent/patients_merged.csv")
merge_df = merge_df.toDF(*[col.lower() for col in merge_df.columns])

merge_df.createOrReplaceTempView("t_merged_patients")
```

- We read the `patients_merged.csv` file prepared in advance for testing the Merge query.
- This file contains 2 duplicate data entries and 2 non-duplicate data entries from the patients.csv file that we previously created as a table.
- In other words, there are a total of 4 data entries, 2 of which are duplicates of the data we previously put into the S3 Tables.
- `merge_df.createOrReplaceTempView("t_merged_patients")` — This part registers the DataFrame read from the CSV file as a Temp View. Once registered, it can be referenced like a table in the Spark SQL syntax.

3. Click the '+' button to add a Cell, and then run the following code to check the data to be merged.

```sql
%%sql
SELECT * FROM t_merged_patients
```

You will see the following data.

![Merge data](/images/workshop/merge_data.webp)

4. Click the '+' button to add a Cell, and then run the following code to check the number of data entries before the merge.

```sql
%%sql
SELECT COUNT(*) FROM s3table.workshop_namespace.patients    
```

You can confirm that there are 3539 entries.

5. Click the '+' button to add a Cell, insert the following code, and Run it.

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

- We perform the Merge query using `spark.sql()`.
- Looking at the Merge query:
  - `s3table.workshop_namespace.patients` is the Target table to merge data into.
  - `t_merged_patients` is the Source table that we previously registered as a Temp View.
  - The part after the `ON` clause sets the matching condition. Since the `id` column is a unique key, we check for duplicates using this column.
  - `WHEN MATCHED THEN` means that for the matching data, the UPDATE statement below is executed.
  - `WHEN NOT MATCHED THEN` means that for the non-matching data, the INSERT statement below is executed.
  - In other words, it is a statement that updates existing data and inserts new data.

6. Click the '+' button to add a Cell, and then run the following code to check the number of data entries after the merge.

```sql
%%sql
SELECT COUNT(*) FROM s3table.workshop_namespace.patients
```

After the merge, you can confirm that there are 3541 entries.
As explained earlier, the data used in the Merge contained a total of 4 data entries, 2 of which were already loaded into the table.
Therefore, the number of rows should only increase by 2.
We confirmed that 3539 + 2 is 3541, and the Merge was performed correctly.

We have merged CSV data into the previously created table.

## Update / Delete Temp Rows

{{% notice info %}}
This section updates or deletes data from the S3 Tables table.
{{% /notice %}}

### Update Temp Rows

{{% notice warning %}}
Please record the time when you execute the update query. It will be used in the snapshot-related exercise.
{{% /notice %}}

1. Click the '+' button to add a Cell, and then Run the following code.

```sql
%%sql
UPDATE s3table.workshop_namespace.patients SET birthdate='2000-01-01' WHERE id='merge_id_1'
```

This updates the birthdate to '2000-01-01' for the row where the id column is 'merge_id_1'.

2. To confirm the updated data, click the '+' button to add a Cell, and then Run the following code.

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.patients WHERE id='merge_id_1'
```

You can confirm that the value of the birthdate column has been updated to '2000-01-01'.

The update has been completed successfully.

### Delete Temp Rows

{{% notice warning %}}
Please record the time when you execute the delete query. It will be used in the snapshot-related exercise.
{{% /notice %}}

1. Click the '+' button to add a Cell, and then Run the following code.

```sql
%%sql
DELETE FROM s3table.workshop_namespace.patients WHERE id IN ('merge_id_1', 'merge_id_2')
```

This deletes the temporarily merged data where the id column value is 'merge_id_1' or 'merge_id_2'.

2. Click the '+' button to add a Cell, and then Run the following code to confirm that the data has been deleted correctly.

```sql
%%sql
SELECT * FROM s3table.workshop_namespace.patients WHERE id IN ('merge_id_1', 'merge_id_2')
```

We confirmed that the data has been deleted correctly.

We have looked at Merge, Update, and Delete queries on the Apache Iceberg table.
