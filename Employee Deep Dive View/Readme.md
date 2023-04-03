``` sql

-- drop cascade required as there is 1 derived view!
DROP VIEW IF EXISTS mv_employees.historic_employee_records CASCADE;
CREATE VIEW mv_employees.historic_employee_records AS
-- we need the previous salary only for the latest record
-- other salary increase/decrease events will use a different field!
WITH cte_previous_salary AS (
  SELECT
    employee_id,
    amount
  FROM (
    SELECT
      employee_id,
      to_date,
      LAG(amount) OVER (
        PARTITION BY employee_id
        ORDER BY from_date
      ) AS amount,
      -- need to rank by descending to get latest record
      ROW_NUMBER() OVER (
        PARTITION BY employee_id
        ORDER BY to_date DESC
      ) AS record_rank
    FROM mv_employees.salary
  ) all_salaries
  -- keep only latest previous_salary records only
  -- must have this in subquery to account for execution order
  WHERE record_rank = 1
),
cte_join_data AS (
SELECT
  employee.id AS employee_id,
  employee.birth_date,
  -- calculated employee_age field
  DATE_PART('year', now()) -
    DATE_PART('year', employee.birth_date) AS employee_age,
  -- employee full name
  CONCAT_WS(' ', employee.first_name, employee.last_name) AS employee_name,
  employee.gender,
  employee.hire_date,
  title.title,
  salary.amount AS salary,
  -- need to separately define the previous_latest_salary
  -- to differentiate between the following lag record!
  cte_previous_salary.amount AS previous_latest_salary,
  department.dept_name AS department,
  -- use the `manager` aliased version of employee table for manager
  CONCAT_WS(' ', manager.first_name, manager.last_name) AS manager,
  -- calculated tenure fields
  DATE_PART('year', now()) -
    DATE_PART('year', employee.hire_date) AS company_tenure_years,
  DATE_PART('year', now()) -
    DATE_PART('year', title.from_date) AS title_tenure_years,
  DATE_PART('year', now()) -
    DATE_PART('year', department_employee.from_date) AS department_tenure_years,
  -- we also need to use AGE & DATE_PART functions here to generate month diff
  DATE_PART('months', AGE(now(), title.from_date)) AS title_tenure_months,
  GREATEST(
    title.from_date,
    salary.from_date,
    department_employee.from_date,
    department_manager.from_date
  ) AS effective_date,
  LEAST(
    title.to_date,
    salary.to_date,
    department_employee.to_date,
    department_manager.to_date
  ) AS expiry_date
FROM mv_employees.employee
INNER JOIN mv_employees.title
  ON employee.id = title.employee_id
INNER JOIN mv_employees.salary
  ON employee.id = salary.employee_id
INNER JOIN mv_employees.department_employee
  ON employee.id = department_employee.employee_id
-- NOTE: department is joined only to the department_employee table!
INNER JOIN mv_employees.department
  ON department_employee.department_id = department.id
-- add in the department_manager information onto the department table
INNER JOIN mv_employees.department_manager
  ON department.id = department_manager.department_id
-- join again on the employee_id field to another employee for manager's info
INNER JOIN mv_employees.employee AS manager
  ON department_manager.employee_id = manager.id
-- join onto our previous cte_previous_salary only for previous_latest_salary
INNER JOIN cte_previous_salary
  ON mv_employees.employee.id = cte_previous_salary.employee_id
),
-- now we apply the window function to order our transactions
-- we will filter out the top 5 in the next CTE step
cte_ordered_transactions AS (
  SELECT
    employee_id,
    birth_date,
    employee_age,
    employee_name,
    gender,
    hire_date,
    title,
    LAG(title) OVER w AS previous_title,
    salary,
    -- previous latest salary is based off the CTE
    previous_latest_salary,
    LAG(salary) OVER w AS previous_salary,
    department,
    LAG(department) OVER w AS previous_department,
    manager,
    LAG(manager) OVER w AS previous_manager,
    company_tenure_years,
    title_tenure_years,
    title_tenure_months,
    department_tenure_years,
    effective_date,
    expiry_date,
    -- we use a reverse ordered effective date window to capture last 5 events
    ROW_NUMBER() OVER (
      PARTITION BY employee_id
      ORDER BY effective_date DESC
    ) AS event_order
  FROM cte_join_data
  -- apply logical filter to remove invalid records resulting from the join
  WHERE effective_date <= expiry_date
  -- define window frame with chronological ordering by effective date
  WINDOW
    w AS (PARTITION BY employee_id ORDER BY effective_date)
),
-- finally we apply our case when statements to generate the employee events
-- and generate our benchmark comparisons for the final output
-- we aliased our FROM table as "base" for compact code!
final_output AS (
  SELECT
    base.employee_id,
    base.gender,
    base.birth_date,
    base.employee_age,
    base.hire_date,
    base.title,
    base.employee_name,
    base.previous_title,
    base.salary,
    -- previous latest salary is based off the CTE
    previous_latest_salary,
    -- previous salary is based off the LAG records
    base.previous_salary,
    base.department,
    base.previous_department,
    base.manager,
    base.previous_manager,
    -- tenure metrics
    base.company_tenure_years,
    base.title_tenure_years,
    base.title_tenure_months,
    base.department_tenure_years,
    base.event_order,
    -- only include the latest salary change for the first event_order row
    CASE
      WHEN event_order = 1
        THEN ROUND(
          100 * (base.salary - base.previous_latest_salary) /
            base.previous_latest_salary::NUMERIC,
          2
        )
      ELSE NULL
    END AS latest_salary_percentage_change,
    -- also include the amount change
    CASE
      WHEN event_order = 1
        THEN ROUND(
          base.salary - base.previous_latest_salary
        )
      ELSE NULL
    END AS latest_salary_amount_change,
    -- event type logic by comparing all of the previous lag records
    CASE
      WHEN base.previous_salary < base.salary
        THEN 'Salary Increase'
      WHEN base.previous_salary > base.salary
        THEN 'Salary Decrease'
      WHEN base.previous_department <> base.department
        THEN 'Dept Transfer'
      WHEN base.previous_manager <> base.manager
        THEN 'Reporting Line Change'
      WHEN base.previous_title <> base.title
        THEN 'Title Change'
      ELSE NULL
    END AS event_name,
    -- salary change
    ROUND(base.salary - base.previous_salary) AS salary_amount_change,
    ROUND(
      100 * (base.salary - base.previous_salary) / base.previous_salary::NUMERIC,
      2
    ) AS salary_percentage_change,
    -- benchmark comparisons - we've omit the aliases for succinctness!
    -- tenure
    ROUND(tenure_benchmark_salary) AS tenure_benchmark_salary,
    ROUND(
      100 * (base.salary - tenure_benchmark_salary)
        / tenure_benchmark_salary::NUMERIC
    ) AS tenure_comparison,
    -- title
    ROUND(title_benchmark_salary) AS title_benchmark_salary,
    ROUND(
      100 * (base.salary - title_benchmark_salary)
        / title_benchmark_salary::NUMERIC
    ) AS title_comparison,
    -- department
    ROUND(department_benchmark_salary) AS department_benchmark_salary,
    ROUND(
      100 * (salary - department_benchmark_salary)
        / department_benchmark_salary::NUMERIC
    ) AS department_comparison,
    -- gender
    ROUND(gender_benchmark_salary) AS gender_benchmark_salary,
    ROUND(
      100 * (base.salary - gender_benchmark_salary)
        / gender_benchmark_salary::NUMERIC
    ) AS gender_comparison,
    -- usually best practice to leave the effective/expiry dates at the end
    base.effective_date,
    base.expiry_date
  FROM cte_ordered_transactions AS base  -- used alias here for the joins below
  INNER JOIN mv_employees.tenure_benchmark
    ON base.company_tenure_years = tenure_benchmark.company_tenure_years
  INNER JOIN mv_employees.title_benchmark
    ON base.title = title_benchmark.title
  INNER JOIN mv_employees.department_benchmark
    ON base.department = department_benchmark.department
  INNER JOIN mv_employees.gender_benchmark
    ON base.gender = gender_benchmark.gender
  -- apply filter to only keep the latest 5 events per employee
  -- WHERE event_order <= 5
)
-- finally we are done with the historic values
SELECT * FROM final_output;

-- This final view powers the employee deep dive tool
-- by keeping only the 5 latest events
DROP VIEW IF EXISTS mv_employees.employee_deep_dive;
CREATE VIEW mv_employees.employee_deep_dive AS
SELECT *
FROM mv_employees.historic_employee_records
WHERE event_order <= 5;

```
