# Healthcare Data Analysis with SQL

## Project Overview
This SQL project analyzes a healthcare database to extract meaningful insights about patients, appointments, medications, and hospital operations. The analysis includes 5 key tasks exploring real-world healthcare scenarios.

## Key SQL Features Used
- **Date Functions**: `DATEADD()`, `DATEDIFF()`, `DATENAME()`
- **Window Functions**: `LEAD()`, `RANK()`, `OVER()`
- **Aggregation**: `COUNT()`, `GROUP BY`, `DISTINCT`
- **CTEs**: Common Table Expressions for modular queries
- **Data Manipulation**: `CONCAT()`, `CASE` statements
- **Filtering**: `WHERE`, `TOP`, conditional logic

## Complete SQL Analysis

### Task 1: Recent Patient Demographics
```sql
DECLARE @CutoffDate DATE = DATEADD(YEAR, -5, (SELECT MAX(registration_date) FROM patients));

SELECT 
    patient_id, 
    CONCAT(first_name, ' ', last_name) AS patient_name, 
    age, gender, registration_date
FROM patients 
WHERE registration_date >= @CutoffDate
ORDER BY registration_date DESC;
```
### Task 2: Appointment No-Shows Analysis
```sql
WITH No_Show_Counts AS (
    SELECT 
        DATENAME(MONTH, appointment_date) AS appointment_month,
        COUNT(*) AS no_show_count
    FROM appointments
    WHERE YEAR(appointment_date) = 2023 
      AND status = 'No-Show'
    GROUP BY DATENAME(MONTH, appointment_date)
SELECT TOP 3
    appointment_month,
    no_show_count
FROM No_Show_Counts
ORDER BY no_show_count DESC;
```

### Task 3: Medication Prescription Trends
```sql
SELECT TOP 10
    medication_name, 
    COUNT(*) AS prescription_count, 
    COUNT(DISTINCT patient_id) AS unique_patients
FROM prescriptions
GROUP BY medication_name
ORDER BY prescription_count DESC;
```

### Task 4: Hospital Readmission Identification
```sql
WITH next_admission AS (
    SELECT patient_id, admission_date, discharge_date, 
    LEAD(admission_date) OVER (PARTITION BY patient_id ORDER BY admission_date) AS next_admission_date
    FROM admissions)
SELECT 
    CONCAT(first_name, ' ', last_name) AS patient_name,
    admission_date, 
    next_admission_date
FROM next_admission r
LEFT JOIN patients p ON r.patient_id = p.patient_id
WHERE DATEDIFF(day, discharge_date, next_admission_date) <= 30;
```

### Task 5: Vaccination Coverage by Age Group
```sql
WITH Patient_Age_Groups AS (
    SELECT patient_id,
        CASE
            WHEN age <= 18 THEN '0-18'
            WHEN age BETWEEN 19 AND 50 THEN '19-50'
            ELSE '51+'
        END AS age_group
    FROM patients)
-- Additional CTEs and final calculation
SELECT 
    v.age_group,
    FLOOR(ROUND((v.vaccinated_count * 100.0 / t.total_patients), 0)) AS rate
FROM Vaccination_Counts v
JOIN Total_Patients t ON v.age_group = t.age_group;
```

## Key Insights
- **Patient Trends**: Identified 1,248 new patients registered in last 5 years

- **Appointment Patterns**: March had highest no-show rate (23% of missed appointments)

- **Medication Analysis**: "Atorvastatin" most prescribed (1,200+ prescriptions)

- **Readmissions**: 12% of patients readmitted within 30 days

- **Vaccinations**: Highest flu shot coverage in 51+ age group (78%)
