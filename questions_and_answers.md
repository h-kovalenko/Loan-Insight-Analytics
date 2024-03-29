## Question 1: High Approval Rates for DTIs >60%
The analysis found that some banks had remarkably high approval rates for applicants with DTIs exceeding 60%. Despite  common practice, suggesting that high DTIs means financial risk, these approvals were unexpected. This finding warrants further investigation into the lending practices of these banks.

![alt text](graphs/Question1.png?raw=true)

*First Bank and Mortgage Research Center have a significantly high loan approval rate for applicants with Debt-to-Income Ratios >60%, far exceeding the 16% baseline rate.*

````sql
WITH
approval_rate AS (
SELECT
b.respondent_name,
l.lei,
COUNT(l.action_taken) AS number_of_approved_loans,
l.debt_to_income_ratio
FROM
hkovalenko.Projectquery.2021_lar l
JOIN
`hkovalenko.Projectquery.Bank names` b
ON
b.lei=l.lei
WHERE
l.action_taken IN (1,
2) AND l.debt_to_income_ratio = '>60%'
GROUP BY
l.lei,
l.debt_to_income_ratio,
b.respondent_name),


total_num AS(
SELECT
lei,
debt_to_income_ratio,
COUNT(debt_to_income_ratio) AS total_num_loans
FROM
hkovalenko.Projectquery.2021_lar
WHERE debt_to_income_ratio = '>60%'
GROUP BY
lei,
debt_to_income_ratio )
SELECT
a.respondent_name,
(a.number_of_approved_loans/t.total_num_loans)*100 AS approval_rate,
a.debt_to_income_ratio, t.total_num_loans
FROM
approval_rate a
JOIN
total_num t
ON
a.lei=t.lei AND
a.debt_to_income_ratio = t.debt_to_income_ratio
WHERE
t.total_num_loans > 1000
````



## Question 2: Racial Disparities in Loan Denial Rates
Racial disparities in loan denial rates were evident in the dataset. While Whites and Asians had relatively consistent denial rates across income groups  Black applicants faced significantly higher denial rates even with similar income levels. This highlights the presence of disparities in mortgage lending.

![alt text](graphs/Question2.png?raw=true)
*Black people within the same income group have a higher denial rate when it comes to home purchases.* 

````sql

WITH RacePurposeCounts AS (
SELECT
CASE derived_race
WHEN 'American Indian or Alaska Native' THEN 'American Indian or Alaska Native'
WHEN 'Asian' THEN 'Asian'
WHEN 'Black or African American' THEN 'Black or African American'
WHEN 'Native Hawaiian or Other Pacific Islander' THEN 'Native Hawaiian or Other Pacific Islander'
WHEN 'White' THEN 'White'
WHEN '2 or more minority races' THEN '2 or more minority races'
WHEN 'Joint' THEN 'Joint'
WHEN 'Free Form Text Only' THEN 'Free Form Text Only'
WHEN 'Race Not Available' THEN 'Race Not Available'
ELSE 'Other'
END AS Race,
CASE loan_purpose
WHEN 1 THEN 'Home purchase'
WHEN 2 THEN 'Home improvement'
WHEN 31 THEN 'Refinancing'
WHEN 32 THEN 'Cash-out refinancing'
WHEN 4 THEN 'Other purpose'
ELSE 'Other'
END AS Purpose,
CASE
WHEN income >= 0 AND income < 50 THEN '0-50K'
WHEN income >= 50 AND income < 75 THEN '50-75K'
WHEN income >= 75 AND income < 100 THEN '75-100K'
WHEN income >= 100 AND income < 150 THEN '100-150K'
WHEN income >= 150 THEN '150K+'
ELSE 'Other'
END AS IncomeGroup,
COUNT(*) AS Count_of_denied
FROM hkovalenko.Projectquery.2021_lar
WHERE derived_race NOT IN ('Race Not Available', 'Free Form Text Only') AND action_taken = 3
GROUP BY Race, Purpose, IncomeGroup
)

, TotalCounts AS (
SELECT
CASE derived_race
WHEN 'American Indian or Alaska Native' THEN 'American Indian or Alaska Native'
WHEN 'Asian' THEN 'Asian'
WHEN 'Black or African American' THEN 'Black or African American'
WHEN 'Native Hawaiian or Other Pacific Islander' THEN 'Native Hawaiian or Other Pacific Islander'
WHEN 'White' THEN 'White'
WHEN '2 or more minority races' THEN '2 or more minority races'
WHEN 'Joint' THEN 'Joint'
WHEN 'Free Form Text Only' THEN 'Free Form Text Only'
WHEN 'Race Not Available' THEN 'Race Not Available'
ELSE 'Other'
END AS Race,
CASE loan_purpose
WHEN 1 THEN 'Home purchase'
WHEN 2 THEN 'Home improvement'
WHEN 31 THEN 'Refinancing'
WHEN 32 THEN 'Cash-out refinancing'
WHEN 4 THEN 'Other purpose'
ELSE 'Other'
END AS Purpose,
CASE
WHEN income >= 0 AND income < 50 THEN '0-50K'
WHEN income >= 50 AND income < 75 THEN '50-75K'
WHEN income >= 75 AND income < 100 THEN '75-100K'
WHEN income >= 100 AND income < 150 THEN '100-150K'
WHEN income >= 150 THEN '150K+'
ELSE 'Other'
END AS IncomeGroup,
COUNT(*) AS total_count
FROM hkovalenko.Projectquery.2021_lar
WHERE derived_race NOT IN ('Race Not Available', 'Free Form Text Only')
GROUP BY Race, Purpose, IncomeGroup
)

SELECT
cte.Race,
cte.Purpose,
cte.IncomeGroup,
cte.Count_of_denied,
total_num.total_count,
ROUND((cte.Count_of_denied/total_num.total_count)*100, 2) as percent_denied

FROM RacePurposeCounts cte
JOIN TotalCounts total_num
ON cte.Race = total_num.Race AND cte.Purpose = total_num.Purpose AND cte.IncomeGroup = total_num.IncomeGroup
WHERE total_num.total_count > 50 AND cte.Purpose = 'Home purchase'
````



## Question 3: Racial Disparities at Different Banks
The analysis further revealed differences in denial rates among various banks, with some institutions having much higher denial rates for Black applicants compared to White applicants. The inclusion of well-known banks emphasized the significance of these disparities.
![alt text](graphs/Question3.png?raw=true)

*Some banks have a much higher denial rate for Black or African American applicants compared to white applicants.* 

```sql
WITH cte AS (
SELECT
lei,
CASE derived_race
WHEN 'White' THEN 'White'
WHEN 'Black or African American' THEN 'Black or African American'
ELSE 'Other'
END AS Race,
CASE loan_purpose
WHEN 1 THEN 'Home purchase'
WHEN 2 THEN 'Home improvement'
WHEN 31 THEN 'Refinancing'
WHEN 32 THEN 'Cash-out refinancing'
WHEN 4 THEN 'Other purpose'
ELSE 'Other'
END AS Purpose,
COUNT(*) AS Count_of_denied
FROM hkovalenko.Projectquery.2021_lar
WHERE derived_race IN ('White', 'Black or African American') AND action_taken = 3
GROUP BY lei, Race, Purpose
)

, total_num AS (
SELECT
lei,
CASE derived_race
WHEN 'White' THEN 'White'
WHEN 'Black or African American' THEN 'Black or African American'
ELSE 'Other'
END AS Race,
CASE loan_purpose
WHEN 1 THEN 'Home purchase'
WHEN 2 THEN 'Home improvement'
WHEN 31 THEN 'Refinancing'
WHEN 32 THEN 'Cash-out refinancing'
WHEN 4 THEN 'Other purpose'
ELSE 'Other'
END AS Purpose,
COUNT(*) AS total_count
FROM hkovalenko.Projectquery.2021_lar
WHERE derived_race IN ('White', 'Black or African American')
GROUP BY lei, Race, Purpose
)

SELECT
cte.lei,
cte.Race,
cte.Purpose,
cte.Count_of_denied,
total_num.total_count,
cte.Count_of_denied/total_num.total_count as percent_denied

FROM cte
JOIN total_num
ON cte.lei = total_num.lei AND cte.Race = total_num.Race AND cte.Purpose = total_num.Purpose
WHERE total_num.total_count > 50
```


## Question 4: Denial Rates by Political Leanings
At the state level, the analysis categorized states based on their political leanings. The findings showed that in red states, mortgage denial rates for Black or African American applicants were 1.5% higher compared to blue states. This information sheds light on the impact of political leanings on racial disparities in mortgage lending.

![alt text](graphs/Question4.png?raw=true)

*Purpose: Home purchase*
*Income:100K-150K*

*The mortgage denial rate for Black or African Americans in red states is 1.5% higher than in blue states, while the denial rate for Asians and Whites remains stable.*

```sql
hkovalenko.Projectquery.Red - Blue states
WITH RacePurposeCounts AS (
SELECT
CASE derived_race
WHEN 'American Indian or Alaska Native' THEN 'American Indian or Alaska Native'
WHEN 'Asian' THEN 'Asian'
WHEN 'Black or African American' THEN 'Black or African American'
WHEN 'Native Hawaiian or Other Pacific Islander' THEN 'Native Hawaiian or Other Pacific Islander'
WHEN 'White' THEN 'White'
WHEN '2 or more minority races' THEN '2 or more minority races'
WHEN 'Joint' THEN 'Joint'
WHEN 'Free Form Text Only' THEN 'Free Form Text Only'
WHEN 'Race Not Available' THEN 'Race Not Available'
ELSE 'Other'
END AS Race,
CASE loan_purpose
WHEN 1 THEN 'Home purchase'
WHEN 2 THEN 'Home improvement'
WHEN 31 THEN 'Refinancing'
WHEN 32 THEN 'Cash-out refinancing'
WHEN 4 THEN 'Other purpose'
ELSE 'Other'
END AS Purpose, state_code,
CASE
WHEN income >= 0 AND income < 50 THEN '0-50K'
WHEN income >= 50 AND income < 75 THEN '50-75K'
WHEN income >= 75 AND income < 100 THEN '75-100K'
WHEN income >= 100 AND income < 150 THEN '100-150K'
WHEN income >= 150 THEN '150K+'
ELSE 'Other'
END AS IncomeGroup,
COUNT(*) AS Count_of_denied
FROM hkovalenko.Projectquery.2021_lar
WHERE derived_race NOT IN ('Race Not Available', 'Free Form Text Only') AND action_taken = 3
GROUP BY Race, Purpose, IncomeGroup, state_code
)

, TotalCounts AS (
SELECT
CASE derived_race
WHEN 'American Indian or Alaska Native' THEN 'American Indian or Alaska Native'
WHEN 'Asian' THEN 'Asian'
WHEN 'Black or African American' THEN 'Black or African American'
WHEN 'Native Hawaiian or Other Pacific Islander' THEN 'Native Hawaiian or Other Pacific Islander'
WHEN 'White' THEN 'White'
WHEN '2 or more minority races' THEN '2 or more minority races'
WHEN 'Joint' THEN 'Joint'
WHEN 'Free Form Text Only' THEN 'Free Form Text Only'
WHEN 'Race Not Available' THEN 'Race Not Available'
ELSE 'Other'
END AS Race,
CASE loan_purpose
WHEN 1 THEN 'Home purchase'
WHEN 2 THEN 'Home improvement'
WHEN 31 THEN 'Refinancing'
WHEN 32 THEN 'Cash-out refinancing'
WHEN 4 THEN 'Other purpose'
ELSE 'Other'
END AS Purpose, state_code,
CASE
WHEN income >= 0 AND income < 50 THEN '0-50K'
WHEN income >= 50 AND income < 75 THEN '50-75K'
WHEN income >= 75 AND income < 100 THEN '75-100K'
WHEN income >= 100 AND income < 150 THEN '100-150K'
WHEN income >= 150 THEN '150K+'
ELSE 'Other'
END AS IncomeGroup,
COUNT(*) AS total_count
FROM hkovalenko.Projectquery.2021_lar
WHERE derived_race NOT IN ('Race Not Available', 'Free Form Text Only')
GROUP BY Race, Purpose, IncomeGroup, state_code
)

SELECT cte.state_code,
cte.Race,
cte.Purpose,
cte.IncomeGroup,
cte.Count_of_denied,
total_num.total_count,
ROUND((cte.Count_of_denied/total_num.total_count)*100, 2) as percent_denied

FROM RacePurposeCounts cte
JOIN TotalCounts total_num
ON cte.Race = total_num.Race AND cte.Purpose = total_num.Purpose AND cte.IncomeGroup = total_num.IncomeGroup AND cte.state_code = total_num.state_code
WHERE total_num.total_count > 50 AND cte.Purpose = 'Home purchase' AND cte.IncomeGroup = "75-100K";
```
Second Query

```sql
SELECT Race, avg(percent_denied) as denial_rate,
CASE
WHEN state_code IN ('AL', 'AK', 'AR', 'ID', 'IN', 'IA', 'KS', 'KY', 'LA', 'MS', 'MO', 'MT', 'NE', 'ND', 'OK', 'SC', 'SD', 'TN', 'TX', 'UT', 'WV', 'WY') THEN 'Red State'
WHEN state_code IN ('CA', 'CO', 'CT', 'DE', 'DC', 'HI', 'IL', 'ME', 'MD', 'MA', 'NH', 'NJ', 'NM', 'NY', 'OR', 'RI', 'VT', 'VA', 'WA') THEN 'Blue State'
ELSE 'Other State'
END AS StateCategory
FROM `hkovalenko.Projectquery.Red - Blue states` group by Race, StateCategory
```


