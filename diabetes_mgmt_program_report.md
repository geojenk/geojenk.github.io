## Diabetes Management Program Report

Author: Georgia Jenkins

Date: 22 July 2024

### Results

**Table 1: Diabetes Onset Date**

Visit histories, A1C lab histories, and recorded onset dates were cross-referenced to determine each patient's most likely onset date.

| Patient ID | Most Likely Onset Date |
|:----------:|:----------------------:|
| 1          | 2024-03-01             |
| 2          | 2023-05-17             |
| 3          | 2010-03-06             |
| 4          | 2023-06-28             |
| 5          | 2023-09-22             |
| 6          | 2021-03-31             |
| 7          | 2021-06-20             |
| 8          | 2023-08-02             |

---

**Table 2: Hemoglobin A1C Lab Results**

Summary of the most recent lab date, result, and change from the previous lab result for each patient. A positive change value indicates an increase from the previous visit. A negative value indicates a decrease. Note that Patient 1 has only one recorded lab visit, and Patient 5's most recent lab result was not recorded. 

| Patient ID | Latest Lab | Latest A1C Result   | Change from Previous <br> (latest - previous) |
|:----------:|:----------:|:-------------------:|:---------------------------------------------:|
| 1          | 2024-03-01 | 9.3                 | NA - no previous lab                          |
| 2          | 2024-03-05 | 8.3                 | 1.4                                           |
| 4          | 2024-03-25 | 7.0                 | -0.2                                          |
| 5          | 2020-10-22 | NA - missing result | NA                                            |
| 6          | 2021-06-30 | 10.2                | -1.0                                          |

---

**Table 3: Total Visits for the Last 2 Years**

The total visit count includes regular visits and A1C lab visits. Data was limited to the last two years (July 2022-July 2024).

| Patient ID | Total Visits |
|:----------:|:------------:|
| 1          | 2            |
| 2          | 4            |
| 4          | 3            |
| 5          | 2            |

### Queries

#### Query 1: Most likely onset date

**Method & Rationale:** To determine the most likely diabetes onset date for each patient, we need to identify the earliest valid date that indicates that the patient is living with diabetes. I took the following approach:

1. Compile each patient's earliest visit date and the earliest date when the patient had an A1C result above 7. 

2. Combine these dates with valid recorded onset dates. To be considered "valid," the onset date could **NOT** be any of the following: 
   
   1. NULL 
   
   2. '1901-01-01' (a common value for missing dates)
   
   3. A date occurring after the record was added to the system (i.e., "in the future")

3. Now we have a set of possible onset dates for each patient. Some patients will have only 1 possible date, while others could have up to 3. 

4. Since all of these dates represent a time when the patient had diabetes, the most likely onset date is the minimum (earliest) date from the set for each patient. 

```sql
--Question 1: Most likely onset date
--Identify the earliest visit date
WITH PossibleDates AS(
  SELECT patient_ID 
  ,MIN(visit_date) AS possible_onset
  FROM diabetes_visits 
  GROUP BY patient_ID
  UNION
--Identify the earliest date of a lab result > 7
  SELECT patient_ID 
  ,MIN(lab_date) AS possible_onset
  FROM a1c_labs
  WHERE CAST(
    REPLACE(result,'..','.') -- may need to add more robust cleaning if other typos exist
    AS DECIMAL(10,1)) > 7
  GROUP BY patient_ID
  UNION
--Identify patient IDs with valid onset date recorded
  SELECT patient_id
  ,onset_date AS possible_onset
  FROM diabetes_cases
  -- may need to add more robust cleaning if other invalid date types exist
  WHERE onset_date IS NOT NULL 
  AND onset_date <= added_date 
  AND onset_date <> '1901-01-01')
  
--Choose the earliest possible_onset date for each patient
  SELECT patient_ID
  ,TO_CHAR(MIN(possible_onset),'YYYY-MM-DD') as most_likely_onset
  FROM PossibleDates
  GROUP BY patient_ID
  ORDER BY patient_ID;
```

#### Query #2: Hemoglobin Results

```sql
--Question 2: latest hemoglobin a1c lab date and result for each patient, along with the delta/difference from their previous one
--Order the lab dates for each patient
WITH OrderedDates AS(
  SELECT 
  patient_ID,
  TO_CHAR(lab_date,'YYYY-MM-DD') AS lab_date, --format dates
  CAST(REPLACE(result, '..', '.') AS DECIMAL(10, 1)) AS lab_result, -- format results as decimals
  ROW_NUMBER() OVER (PARTITION BY patient_ID ORDER BY lab_date DESC) AS row_num -- number rows by visit date for each patient
  FROM a1c_labs)
  
  SELECT
  latest.patient_ID,
  latest.lab_date AS latest_lab,
  latest.lab_result AS latest_result,
  --Optional: uncomment below to include previous lab data
  --previous.lab_date AS previous_lab,
  --previous.lab_result AS previous_result,
  latest.lab_result - previous.lab_result AS result_change
  FROM OrderedDates latest
  LEFT JOIN OrderedDates previous 
  ON latest.patient_ID = previous.patient_ID 
  AND previous.row_num = 2
  WHERE latest.row_num = 1
  ORDER BY latest.patient_ID;
```

#### Query #3: Total Visits

```sql
-- Question 3: Total visits per patient over the last 2 years
WITH AllVisits AS(
  SELECT *
  FROM diabetes_visits 
  WHERE visit_date >= CURRENT_DATE - INTERVAL '2 years'
  UNION
--Count labs as visit dates
  SELECT patient_ID
  ,lab_date as visit_date
  FROM a1c_labs
  WHERE lab_date >= CURRENT_DATE - INTERVAL '2 years')
  
  SELECT patient_ID
  ,COUNT(visit_date) AS total_visits
  FROM AllVisits
  GROUP BY patient_ID
  ORDER BY patient_ID;
```



---

[View on DB Fiddle](https://www.db-fiddle.com/f/f8N1M9Tdqm6UJWstfPd3au/2#)
