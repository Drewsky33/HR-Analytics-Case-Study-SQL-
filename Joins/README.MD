``` sql

-- Straight inner join

DROP TABLE IF EXISTS naive_join_table;
CREATE TEMP TABLE naive_join_table AS
SELECT
  employee.id,
  employee.birth_date,
  employee.first_name,
  employee.last_name,
  employee.gender,
  employee.hire_date,
  -- we do not need title.employee_id as employee.id is already included!
  title.title,
  title.from_date AS title_from_date,
  title.to_date AS title_to_date,
  -- same goes for the title.employee_id column
  salary.amount,
  salary.from_date AS salary_from_date,
  salary.to_date AS salary_to_date,
  -- same for department_employee.employee_id
  -- shorten department_employee to dept for the aliases
  department_employee.department_id,
  department_employee.from_date AS dept_from_date,
  department_employee.to_date AS dept_to_date,
  -- we do not need department.department_id as it is already included!
  department.dept_name
FROM mv_employees.employee
INNER JOIN mv_employees.title
  ON employee.id = title.employee_id
INNER JOIN mv_employees.salary
  ON employee.id = salary.employee_id
INNER JOIN mv_employees.department_employee
  ON employee.id = department_employee.employee_id
-- NOTE: department is joined only to the department_employee table!
INNER JOIN mv_employees.department
  ON department_employee.department_id = department.id;
  
-- Inspecting table

SELECT
  COUNT(*) AS row_count
FROM naive_join_table;


There seems to be a difference between the original table and the counts for this new table created from joining information. I'm going to inspect individuals and try to see what's going on. 

-- Inspecting individuals
SELECT *
FROM naive_join_table
WHERE id = 10001
ORDER BY salary_to_date DESC;

There's a little too much information in this view. I'm going to have a look at another employee.

-- Look at another employee
SELECT *
FROM naive_join_table
WHERE id = 11669
ORDER BY salary_to_date DESC;

It looks like there are some duplicates happening here. I'm going to zoom in on the salary_to_date column and see what's going on. 

-- Let's look further
SELECT *
FROM naive_join_table
WHERE id = 11669
  AND salary_to_date = '9999-01-01';
  
 It looks like the employee has been listed in two different apartments. Additionally, the employee has two different titles at the same time. This is due to the date information creating duplicate entries in the table. 
 
 -- Filter for the information to be the same across the table. 
 SELECT *
FROM naive_join_table
WHERE id = 11669
  AND salary_to_date = '9999-01-01'
  AND title_to_date = '9999-01-01'
  AND dept_to_date = '9999-01-01';
  
 
 So that where filter seems to have worked. We'll incorporate that into the join stage so that we won't have duplicate entries. 
 
 -- Creating the current join table
DROP TABLE IF EXISTS current_join_table;
CREATE TEMP TABLE current_join_table AS 
SELECT
  employee.id,
  employee.birth_date,
  employee.first_name,
  employee.last_name,
  employee.gender,
  employee.hire_date,
  -- We don't need employee id as it's already included in the employee table
  title.title,
  title.from_date AS title_from_date,
  title.to_date AS title_to_date,
  -- Still don't need the employee id from the salary table
  salary.amount,
  salary.from_date AS salary_from_date,
  salary.to_date AS salary_to_date,
  -- Same for department employee table
  department_employee.department_id,
  department_employee.from_date AS dept_from_date,
  department_employee.to_date AS dept_to_date,
  -- department id already included so we just grab the name
  department.dept_name
FROM mv_employees.employee
INNER JOIN mv_employees.title
  ON employee.id = title.employee_id
INNER JOIN mv_employees.salary
  ON employee.id = salary.employee_id
INNER JOIN mv_employees.department_employee
  ON employee.id = department_employee.employee_id
-- Note: department is joined only to the department employee table
INNER JOIN mv_employees.department
  ON department_employee.department_id = department.id
-- Don't want to use a CTE for the filter step
WHERE salary.to_date = '9999-01-01'
  AND title.to_date = '9999-01-01'
  AND department_employee.to_date = '9999-01-01';
  
  -- Let's look at the counts to see if it matches the values in the original table
SELECT
  COUNT(*) AS row_count
FROM current_join_table;


It looks like the information is what we want.
  
