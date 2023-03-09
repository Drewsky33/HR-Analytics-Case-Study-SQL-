``` sql
DROP SCHEMA IF EXISTS adjusted_employees CASCADE;
CREATE SCHEMA adjusted_employees;

-- employee
DROP TABLE IF EXISTS adjusted_employees.employee;
CREATE TABLE adjusted_employees.employee AS
SELECT * FROM employees.employee;

-- department employee
DROP TABLE IF EXISTS adjusted_employees.department;
CREATE TABLE adjusted_employees.department AS
SELECT * FROM employees.department;

-- department employee
DROP TABLE IF EXISTS adjusted_employees.department_employee;
CREATE TABLE adjusted_employees.department_employee AS
SELECT * FROM employees.department_employee;

-- department manager
DROP TABLE IF EXISTS adjusted_employees.department_manager;
CREATE TABLE adjusted_employees.department_manager AS
SELECT * FROM employees.department_manager;

-- salary
DROP TABLE IF EXISTS adjusted_employees.salary;
CREATE TABLE adjusted_employees.salary AS
SELECT * FROM employees.salary;

-- title
DROP TABLE IF EXISTS adjusted_employees.title;
CREATE TABLE adjusted_employees.title AS
SELECT * FROM employees.title;

-- update employee
UPDATE adjusted_employees.employee
SET hire_date = hire_date + INTERVAL '18 YEARS';

UPDATE adjusted_employees.employee
SET birth_date = birth_date + INTERVAL '18 YEARS';

-- update adjusted_employees.department_employee
UPDATE adjusted_employees.department_employee SET
from_date = from_date + INTERVAL '18 YEARS';

UPDATE adjusted_employees.department_employee
SET to_date = to_date + INTERVAL '18 YEARS'
WHERE to_date <> '9999-01-01';

-- update department_manager
UPDATE adjusted_employees.department_manager
SET from_date = from_date + INTERVAL '18 YEARS';

UPDATE adjusted_employees.department_manager
SET to_date = to_date + INTERVAL '18 YEARS'
WHERE to_date <> '9999-01-01';

-- update salary
UPDATE adjusted_employees.salary
SET from_date = from_date + INTERVAL '18 YEARS';

UPDATE adjusted_employees.salary
SET to_date = to_date + INTERVAL '18 YEARS'
WHERE to_date <> '9999-01-01';

-- update title
UPDATE adjusted_employees.title
SET from_date = from_date + INTERVAL '18 YEARS';

UPDATE adjusted_employees.title
SET to_date = to_date + INTERVAL '18 YEARS'
WHERE to_date <> '9999-01-01';

```
