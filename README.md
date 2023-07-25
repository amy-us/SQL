# SQL
Basic Retention Curve: 
evaluate the quality of user acquisition in Jan 2019 by the retention metric. 
First, you need to know how many users are retained in each subsequent month from the first month (Jan 2019) 
they pay the successful transaction (only get data of 2019). 

WITH jan_customers AS (
SELECT DISTINCT customer_id
FROM fact_transaction_2019 AS fact_2019
LEFT JOIN dim_scenario AS sce
  ON fact_2019.scenario_id= sce.scenario_id
WHERE sub_category = 'Telco card'
  AND status_id = 1
  AND MONTH (transaction_time) = 1
)
, full_customers AS (
SELECT DISTINCT customer_id
       , MONTH (transaction_time) AS month
FROM fact_transaction_2019 AS fact_2019
LEFT JOIN dim_scenario AS sce
  ON fact_2019.scenario_id= sce.scenario_id
WHERE sub_category = 'Telco card'
AND status_id = 1
)
, join_table AS (
SELECT jan_customers.customer_id, month
FROM jan_customers 
LEFT JOIN  full_customers 
  ON jan_customers.customer_id = full_customers.customer_id
)
SELECT DISTINCT (month - 1) AS subsequent_month 
      , COUNT (customer_id) AS retained_users
FROM join_table
GROUP BY month


́**realize that the number of retained customers has decreased over time. 
**Let’s calculate retention =  number of retained customers / total users of the first month. 

WITH jan_customers AS (
SELECT DISTINCT customer_id
FROM fact_transaction_2019 AS fact_2019
LEFT JOIN dim_scenario AS sce
  ON fact_2019.scenario_id= sce.scenario_id
WHERE sub_category = 'Telco card'
  AND status_id = 1
  AND MONTH (transaction_time) = 1
)
, full_customers AS (
SELECT DISTINCT customer_id
       , MONTH (transaction_time) AS month
FROM fact_transaction_2019 AS fact_2019
LEFT JOIN dim_scenario AS sce
  ON fact_2019.scenario_id= sce.scenario_id
WHERE sub_category = 'Telco card'
AND status_id = 1
)
, join_table AS (
SELECT jan_customers.customer_id, month
FROM jan_customers 
LEFT JOIN  full_customers 
  ON jan_customers.customer_id = full_customers.customer_id
)
SELECT DISTINCT (month - 1) AS subsequent_month 
      , COUNT (customer_id) AS retained_users
      , (SELECT COUNT (DISTINCT customer_id) FROM jan_customers) AS original_users
      , FORMAT (COUNT (customer_id)*1.0/(SELECT COUNT (DISTINCT customer_id) FROM jan_customers), 'p') AS ptc_retained
FROM join_table
GROUP BY month



