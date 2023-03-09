``` sql

-- Use views to access data a certain point in time 
DROP SCHEMA IF EXISTS v_employees CASCADE;
CREATE SCHEMA v_employees;

-- department
DROP VIEW IF EXISTS v_employee.department;
CREATE VIEW v_employees.department AS 
SELECT * FROM employees.department;

-- department employee
DROP VIEW IF EXISTS v_employees.department_employee;
CREATE VIEW v_employees.department_employee AS 
SELECT
  employee_id,
  department_id,
  from_date + INTERVAL '18 YEARS' AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN to_date + interval '18 YEARS'
    ELSE to_date
    END AS to_date
FROM employees.department_employee;

-- department manager
DROP VIEW IF EXISTS v_employees.department_manager;
CREATE VIEW v_employees.department_manager AS
SELECT
  employee_id,
  department_id,
  from_date + interval '18 years' AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN to_date + interval '18 years'
    ELSE to_date
    END AS to_date
FROM employees.department_manager;

-- employee
DROP VIEW IF EXISTS v_employees.employee;
CREATE VIEW v_employees.employee AS
SELECT
  id,
  birth_date + interval '18 years' AS birth_date,
  first_name,
  last_name,
  gender,
  hire_date + interval '18 years' AS hire_date
FROM employees.employee;

-- salary
DROP VIEW IF EXISTS v_employees.salary;
CREATE VIEW v_employees.salary AS
SELECT
  employee_id,
  amount,
  from_date + interval '18 years' AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN to_date + interval '18 years'
    ELSE to_date
    END AS to_date
FROM employees.salary;

-- title
DROP VIEW IF EXISTS v_employees.title;
CREATE VIEW v_employees.title AS
SELECT
  employee_id,
  title,
  from_date + interval '18 years' AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN to_date + interval '18 years'
    ELSE to_date
    END AS to_date
FROM employees.title;
```

``` sql
-- Select
SELECT *
FROM v_employees.salary
WHERE employee_id = 10001
ORDER BY from_date DESC
LIMIT 5;

```
