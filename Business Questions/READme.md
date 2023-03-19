We have a few requirements for the dashboard. Before we get to that, we'll answer some questions from the table. 

``` sql

-- What is ther average salary of someone from production department
SELECT
  AVG(amount) AS average_salary
FROM current_join_table
WHERE dept_name = 'Production';

-- Which position has the highest average salary
SELECT
  title, 
  AVG(amount) AS avg_salary
FROM current_join_table
GROUP BY title
ORDER BY avg_salary DESC;

-- Which department has the lowest average salary
SELECT
  dept_name,
  AVG(amount) AS avg_salary
FROM current_join_table
GROUP BY dept_name
ORDER BY avg_salary;

-- Calculating average salary increases
WITH lag_data AS (
SELECT
  employee_id,
  to_date,
  amount AS current_amount,
  LAG(amount) OVER (PARTITION BY employee_id ORDER BY to_date) AS previous_amount
FROM mv_employees.salary
WHERE employee_id = 10001
)

SELECT
  employee_id,
  current_amount - previous_amount AS salary_amount_change,
  100 * (current_amount - previous_amount) / previous_amount::NUMERIC AS salary_pc_change
FROM lag_data
WHERE to_date = '9999-01-01';

```
