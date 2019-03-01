<!---
Group:condition era
Name:CE03 Min/max, average length of condition stratified by age/gender
Author:Patrick Ryan
CDM Version: 5.0
-->

# CE03: Min/max, average length of condition stratified by age/gender

## Description
## Input

|  Parameter |  Example |  Mandatory |  Notes |
| --- | --- | --- | --- |
| concept_name | OMOP Hip Fracture 1 |  Yes |  concept_id=500000601 |

## Query
The following is a sample run of the query. The input parameters are highlighted in blue

```sql
WITH hip_fracture  AS (
SELECT DISTINCT descendant_concept_id 
  FROM @vocab.relationship r
  JOIN @vocab.concept_relationship cr 
    ON r.relationship_id  = cr.relationship_id 
  JOIN @vocab.concept c1 
    ON c1.concept_id = cr.concept_id_1 
  JOIN @vocab.concept_ancestor ca
    ON ca.ancestor_concept_id = cr.concept_id_2 
 WHERE r.relationship_name = 'HOI contains SNOMED (OMOP)'
   AND c1.concept_name     = 'OMOP Hip Fracture 1' 
), people_with_hip_fracture AS (
SELECT DISTINCT 
       p.person_id, 
       c.concept_name AS gender, 
       YEAR(ce.condition_era_start_date) - p.year_of_birth AS age, 
       ce.condition_era_end_date - ce.condition_era_start_date + 1 AS duration, 
       (YEAR(ce.condition_era_start_date) - p.year_of_birth)/10 AS age_grp 
  FROM @cdm.condition_era ce 
  JOIN hip_fracture hf  
    ON hf.descendant_concept_id = ce.condition_concept_id 
  JOIN @cdm.person p
    ON p.person_id = ce.person_id 
  JOIN @vocab.concept c 
    ON c.concept_id = p.gender_concept_id 
)
SELECT gender, 
       CASE 
         WHEN age_grp = 0 THEN '0-9' 
         WHEN age_grp = 1 THEN '10-19' 
         WHEN age_grp = 2 THEN '20-29' 
         WHEN age_grp = 3 THEN '30-39' 
         WHEN age_grp = 4 THEN '40-49' 
         WHEN age_grp = 5 THEN '50-59' 
         WHEN age_grp = 6 THEN '60-69' 
         WHEN age_grp = 7 THEN '70-79' 
         WHEN age_grp = 8 THEN '80-89' 
         WHEN age_grp = 9 THEN '90-99' 
         WHEN age_grp > 9 THEN '100+' 
       END           AS age_grp, 
       COUNT(*)      AS num_patients, 
       MIN(duration) AS min_duration_count, 
       MAX(duration) AS max_duration_count, 
       AVG(duration) AS avg_duration_count 
  FROM people_with_hip_fracture
 GROUP BY gender, age_grp 
 ORDER BY gender, age_grp;
```


## Output

## Output field list

|  Field |  Description |
| --- | --- |
| gender | Patient gender name. i.e. MALE, FEMALE... |
| age_grp | Age group in increments of 10 years |
| num_patients | Number of patients withing gender and age group with associated condition |
| min_duration | Minimum duration of condition in days |
| max_duration | Maximum duration of condition in days |
| avg_duration | Average duration of condition in days |

## Sample output record

|  Field |  Description |
| --- | --- |
| gender |  FEMALE |
| age_grp |  10-19 |
| num_patients |  518 |
| min_duration |  1 |
| max_duration | 130  |
| avg_duration |  8 |

## Documentation
https://github.com/OHDSI/CommonDataModel/wiki/