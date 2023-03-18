```sql

-- Materialized View Script
DROP SCHEMA IF EXISTS mv_employees CASCADE;
CREATE SCHEMA mv_employees;

-- department
DROP MATERIALIZED VIEW IF EXISTS mv_employees.department;
CREATE MATERIALIZED VIEW mv_employees.department AS
SELECT *
FROM employees.department;

-- department employees
DROP MATERIALIZED VIEW IF EXISTS mv_employees.department_employee;
CREATE MATERIALIZED VIEW mv_employees.department_employee AS
SELECT
  employee_id, 
  department_id,
  (from_date + INTERVAL '18 YEARS')::DATE AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN (to_date + INTERVAL '18 YEARS')::DATE
    ELSE to_date
    END AS to_date
FROM employees.department_employee;

-- Department manager 
DROP MATERIALIZED VIEW IF EXISTS mv_employees.department_manager;
CREATE MATERIALIZED VIEW mv_employees.department_manager AS
SELECT
  employee_id,
  department_id,
  (from_date + INTERVAL '18 YEARS')::DATE AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN (to_date + INTERVAL '18 YEARS')::DATE
    ELSE to_date
    END AS to_date
FROM employees.department_manager;

-- Employee
DROP MATERIALIZED VIEW IF EXISTS mv_employees.employee;
CREATE MATERIALIZED VIEW mv_employees.employee AS
SELECT
  id,
  (birth_date + INTERVAL '18 YEARS')::DATE AS birth_date,
  first_name,
  last_name,
  gender,
  (hire_date + INTERVAL '18 YEARS')::DATE AS hire_date
FROM employees.employee;

-- Salary
DROP MATERIALIZED VIEW IF EXISTS mv_employees.salary;
CREATE MATERIALIZED VIEW mv_employees.salary AS
SELECT
  employee_id,
  amount,
  (from_date + INTERVAL '18 YEARS')::DATE AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN (to_date + INTERVAL '18 YEARS')::DATE
    ELSE to_date
    END AS to_date
FROM employees.salary;

-- title
DROP MATERIALIZED VIEW IF EXISTS mv_employees.title;
CREATE MATERIALIZED VIEW mv_employees.title AS 
SELECT
  employee_id,
  title,
  (from_date + INTERVAL '18 YEARS')::DATE AS from_date,
  CASE
    WHEN to_date <> '9999-01-01' THEN (to_date + INTERVAL '18 YEARS')::DATE
    ELSE to_date
    END AS to_date
FROM employees.title;

-- Creating Indexes
CREATE UNIQUE INDEX ON mv_employees.employee USING btree (id);
CREATE UNIQUE INDEX ON mv_employees.department_employee USING btree (employee_id, department_id);
CREATE INDEX        ON mv_employees.department_employee USING btree (department_id);
CREATE UNIQUE INDEX ON mv_employees.department USING btree (id);
CREATE UNIQUE INDEX ON mv_employees.department USING btree (dept_name);
CREATE UNIQUE INDEX ON mv_employees.department_manager USING btree (employee_id, department_id);
CREATE INDEX        ON mv_employees.department_manager USING btree (department_id);
CREATE UNIQUE INDEX ON mv_employees.salary USING btree (employee_id, from_date);
CREATE UNIQUE INDEX ON mv_employees.title USING btree (employee_id, title, from_date);

```
