# Seasonal Flu Vaccination - Dashboard

## Problem Statement
This Tableau project delves into the detailed analysis and visualization of seasonal flu vaccine administration throughout the year 2022 across various counties in Massachusetts, USA. The main objective is to uncover insights into vaccination patterns among different demographic groups and to monitor the flu vaccination uptake throughout the year among active patients. The findings aim to aid public health officials and the public in understanding the reach and dispersion of flu vaccinations.

Note: All patient data used in this project was syntheitcally generated via Synthea, a synthetic patient generator.

## Data Preparation in SQL

### 1. Active Patients Criteria
To ensure the analysis accurately reflects the present and pertinent patient data, only "active patients" are accounted for. These patients were alive as of December 31, 2022, and aged at least six months. The active_patients Common Table Expression (CTE) was employed for this purpose, filtering patient records from the database using the criteria provided.

```
with active_patients as
(	select distinct patient
	from encounters as e
	join patients as pat
	  on e.patient = pat.id
	where start between '2020-01-01 00:00' and '2022-12-31 23:59'
	  and pat.deathdate is null
	  and EXTRACT(EPOCH FROM age('2022-12-31',pat.birthdate)) / 2592000 >= 6
)
```

### 2. Identifying Flu Vaccinations in 2022
Additionally, a second CTE named flu_shot_2022 was used to identify patients who received the seasonal Flu Vaccine (coded '5302') within the year 2022, recording the earliest date of administration.
```
, flu_shot_2022 as
(
select patient, min(date) as earliest_flu_shot_2022 
from immunizations
where code = '5302'
  and date between '2022-01-01 00:00' and '2022-12-31 23:59'
group by patient
)
```

### 3. Data Merge
The final SQL query merges the 'patients' table with the 'flu_shot_2022' data using a left join. This ensures inclusion of all active patients, regardless of their flu shot status in 2022. The query retrieves comprehensive patient information, such as birthdate, race, county, and age as of the end of 2022, and determines whether each patient received a flu shot (indicated by the flu_shot_2022 binary flag).

```
select pat.birthdate
      ,pat.race
	  ,pat.county
	  ,pat.id
	  ,pat.first
	  ,pat.last
	  ,pat.gender
	  ,extract(YEAR FROM age('2022-12-31', birthdate)) as age
	  ,flu.earliest_flu_shot_2022
	  ,flu.patient
	  ,case when flu.patient is not null then 1 
	   else 0
	   end as flu_shot_2022
from patients as pat
left join flu_shot_2022 as flu
  on pat.id = flu.patient
where 1=1
  and pat.id in (select patient from active_patients)
```

## Data Visualization in SQL
After the data preparation phase, the data is loaded into Tableau by establishing a connection to the PostgreSQL server. This dataset is then leveraged to construct a detailed Tableau dashboard with the following specific objectives:

1. Evaluate the percentage of patients who received their seasonal flu vaccine, segmented by key demographics:
- Age
- Race
- County, with an emphasis on geographical visualization
- Overall compliance rate across the patient population

Furthermore, the dashboard is configured to:
1. Display the running total of flu vaccine administrations during 2022, offering insight into vaccination efforts over the year
2. Present a detailed patient registry that reveals each person's seasonal flu vaccination status in 2022

 # Report Snapshot
 
![Dashboard_upload](https://github.com/Pearlyn-B/da-portfolio/assets/80374547/10b16968-850b-45d6-aefd-7740fb1dd8ba)


