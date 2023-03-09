-- Inspect a few rows
``` sql
SELECT
  *
FROM employees.employee
LIMIT 5;

```

``` sql
-- 3.2 Look at the title table 
SELECT
  employee_id,
  COUNT(*)
FROM employees.title
GROUP BY employee_id
ORDER BY COUNT(*) DESC
LIMIT 10;
```

``` sql
-- Look at an employee with multiple titles 
SELECT *
FROM employees.title
WHERE employee_id = 10612
ORDER BY from_date;
```

```sql
-- Slow changing dimensions are dates from to 2

-- 3.3 Salary table 
SELECT *
FROM employees.salary
WHERE employee_id = 10612
ORDER BY from_date;
```

```sql
-- 3.4 Deparment Employee Table
SELECT *
FROM employees.department_employee
WHERE employee_id = 10612
ORDER BY from_date;
```

```sql
-- Explore employee who has been in the most departments
SELECT
  employee_id,
  COUNT(DISTINCT department_id) AS unique_departments
FROM employees.department_employee
GROUP BY employee_id
ORDER BY unique_departments DESC
LIMIT 5;
```

```sql
-- Look at the first employee to observe the departments he worked in and when they moved 
SELECT *
FROM employees.department_employee
WHERE employee_id = 10040
ORDER BY from_date;
```

```sql
-- 3.5 Department Manager table shows relationship between employees and their respective depts over time.
SELECT *
FROM employees.department_manager
WHERE department_id = 'd003'
ORDER BY from_date;
```

```sql
-- 3.6 department: 1:1 unique relationsip between id or department id and department name
SELECT *
FROM employees.department
ORDER BY id;
```

