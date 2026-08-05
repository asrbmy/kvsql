# SQL Practice Script

Run these blocks top to bottom in the editor (paste the whole file, or one section
at a time — either works). Every command below is confirmed to run in this editor.

---

## 1. Database & Table Setup

```sql
CREATE DATABASE test_db;
USE test_db;

CREATE TABLE departments (
  dept_id   INTEGER PRIMARY KEY AUTO_INCREMENT,
  dept_name VARCHAR(50) NOT NULL UNIQUE
);

CREATE TABLE employees (
  id          INT PRIMARY KEY,
  name        VARCHAR(50) NOT NULL,
  department  VARCHAR(30),
  city        VARCHAR(30),
  salary      INT,
  hire_date   DATE,
  status      ENUM('active','on_leave','terminated') DEFAULT 'active',
  dept_id     INT,
  FOREIGN KEY (dept_id) REFERENCES departments(dept_id) ON DELETE CASCADE
);

CREATE TABLE projects (
  project_id   INTEGER PRIMARY KEY AUTO_INCREMENT,
  project_name VARCHAR(50) NOT NULL,
  emp_id       INT,
  budget       DECIMAL(10,2) CHECK (budget >= 0)
);

ALTER TABLE employees ADD COLUMN bonus INT;
```

---

## 2. INSERT Data

> One employee record intentionally contains a `NULL` city to demonstrate `IS NULL`.

```sql
INSERT INTO departments (dept_id, dept_name) VALUES
  (1, 'Sales'),
  (2, 'Engineering'),
  (3, 'Support');

INSERT INTO employees (id, name, department, city, salary, hire_date, dept_id) VALUES
  (1, 'Ariana Cole',   'Sales',       'Austin',  52000, '2023-01-15', 1),
  (2, 'Devon Marsh',   'Engineering', 'Denver',  91000, '2022-06-21', 2),
  (3, 'Priya Nair',    'Sales',       NULL,      38000, '2024-03-02', 1),
  (4, 'Liam Ortega',   'Support',     'Boise',   27000, '2021-11-09', 3),
  (5, 'Nadia Farrow',  'Engineering', 'Denver', 105000, '2020-08-30', 2);

INSERT INTO projects (project_name, emp_id, budget) VALUES
  ('Website Revamp',   1, 15000.00),
  ('API Migration',    2, 42000.50),
  ('Support Overhaul', 4, 8000.00);
```

---

## 3. SELECT / FROM / WHERE

```sql
SELECT * FROM employees
WHERE salary > 50000;
```

---

## 4. Comparison Operators

```sql
SELECT name, salary FROM employees WHERE salary = 91000;
SELECT name, salary FROM employees WHERE salary <> 91000;
SELECT name, salary FROM employees WHERE salary != 91000;
SELECT name, salary FROM employees WHERE salary < 40000;
SELECT name, salary FROM employees WHERE salary > 40000;
SELECT name, salary FROM employees WHERE salary <= 27000;
SELECT name, salary FROM employees WHERE salary >= 91000;
```

---

## 5. BETWEEN / NOT BETWEEN

```sql
SELECT name, salary
FROM employees
WHERE salary BETWEEN 30000 AND 90000;

SELECT name, salary
FROM employees
WHERE salary NOT BETWEEN 30000 AND 90000;
```

---

## 6. Logical Operators

### AND

```sql
SELECT name, department, salary
FROM employees
WHERE department = 'Sales'
AND salary > 40000;
```

### OR

```sql
SELECT name, department
FROM employees
WHERE department = 'Support'
OR department = 'Sales';
```

### NOT

```sql
SELECT name
FROM employees
WHERE NOT department = 'Engineering';
```

---

## 7. IN / NOT IN

```sql
SELECT name, city
FROM employees
WHERE city IN ('Austin', 'Denver');

SELECT name, city
FROM employees
WHERE city NOT IN ('Austin', 'Denver');
```

---

## 8. LIKE

```sql
SELECT name
FROM employees
WHERE name LIKE 'A%';

SELECT name
FROM employees
WHERE name LIKE '%a';
```

---

## 9. IS NULL / IS NOT NULL

### IS NULL

```sql
SELECT name
FROM employees
WHERE city IS NULL;
```

### IS NOT NULL

```sql
SELECT name
FROM employees
WHERE city IS NOT NULL;
```

---

## 10. DISTINCT

```sql
SELECT DISTINCT department
FROM employees;
```

---

## 11. ORDER BY / ASC / DESC

```sql
SELECT name, salary FROM employees ORDER BY salary;
SELECT name, salary FROM employees ORDER BY salary ASC;
SELECT name, salary FROM employees ORDER BY salary DESC;
SELECT name, department, salary FROM employees ORDER BY department ASC, salary DESC;
```

---

## 12. LIMIT

```sql
SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 2;

SELECT name, salary
FROM employees
ORDER BY salary DESC
LIMIT 2 OFFSET 1;
```

---

## 13. GROUP BY / HAVING

```sql
SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department;

SELECT department, SUM(salary) AS total_salary
FROM employees
GROUP BY department
HAVING SUM(salary) > 60000;
```

---

## 14. Joins & Table Aliases

```sql
-- CROSS JOIN
SELECT e.name, p.project_name
FROM employees e CROSS JOIN projects p;

-- Cartesian product (implicit)
SELECT e.name, p.project_name
FROM employees e, projects p;

-- EQUI JOIN (implicit, via WHERE)
SELECT e.name, d.dept_name
FROM employees e, departments d
WHERE e.dept_id = d.dept_id;

-- INNER JOIN
SELECT e.name, d.dept_name
FROM employees e
JOIN departments d ON e.dept_id = d.dept_id;

-- NATURAL JOIN (matches on same-named columns, here dept_id)
SELECT name, dept_name
FROM employees NATURAL JOIN departments;

-- LEFT JOIN
SELECT e.name, p.project_name
FROM employees e
LEFT JOIN projects p ON e.id = p.emp_id;

-- RIGHT JOIN
SELECT e.name, p.project_name
FROM employees e
RIGHT JOIN projects p ON e.id = p.emp_id;

-- USING
SELECT e.name, d.dept_name
FROM employees e JOIN departments d USING (dept_id);
```

---

## 15. UNION / UNION ALL

```sql
SELECT city FROM employees
UNION
SELECT dept_name FROM departments;

SELECT department FROM employees
UNION ALL
SELECT dept_name FROM departments;
```

---

## 16. Subqueries

```sql
SELECT name
FROM employees
WHERE id IN (SELECT emp_id FROM projects WHERE budget > 10000);
```

---

## 17. CASE WHEN

```sql
SELECT
  name,
  salary,
  CASE
    WHEN salary >= 90000 THEN 'Senior'
    WHEN salary >= 40000 THEN 'Mid'
    ELSE 'Junior'
  END AS band
FROM employees;
```

---

## 18. WITH (Common Table Expression)

```sql
WITH high_earners AS (
  SELECT * FROM employees WHERE salary > 50000
)
SELECT name, department FROM high_earners;
```

---

## 19. Mathematical Functions

```sql
SELECT
  POWER(2, 5)          AS power_result,
  MOD(17, 5)            AS mod_result,
  ROUND(91234.5678, 2)  AS rounded,
  ABS(-42)              AS absolute,
  CEIL(4.2)             AS ceiling,
  FLOOR(4.8)            AS floored,
  SIGN(-20)             AS sign_result,
  SQRT(144)             AS square_root,
  TRUNCATE(15.789, 2)   AS truncated;
```

---

## 20. String Functions

```sql
SELECT
  CHAR(65)            AS char_result,
  CONCAT(name, ' — ', department) AS full_label,
  UCASE(name)          AS ucase_name,
  UPPER(name)          AS upper_name,
  LCASE(department)    AS lcase_dept,
  LOWER(department)    AS lower_dept,
  MID(name, 1, 3)      AS mid_test,
  SUBSTRING(name, 3)   AS substring_test,
  SUBSTR(name, 1, 4)   AS substr_test,
  LENGTH(name)         AS name_length,
  LEFT(name, 4)        AS left_chars,
  RIGHT(name, 4)       AS right_chars,
  INSTR(name, 'a')     AS position_of_a,
  TRIM('  padded  ')   AS trimmed,
  LTRIM('  left')      AS left_trimmed,
  RTRIM('right  ')     AS right_trimmed
FROM employees;
```

---

## 21. Date & Time Functions

```sql
SELECT
  NOW()                       AS current_datetime,
  SYSDATE()                   AS sys_datetime,
  CURDATE()                   AS current_date_string,
  CURRENT_DATE()               AS current_date_literal,
  DATE(hire_date)              AS date_only,
  MONTH(hire_date)             AS hire_month,
  MONTHNAME(hire_date)         AS hire_month_name,
  YEAR(hire_date)              AS hire_year,
  DAY(hire_date)                AS hire_day,
  DAYOFMONTH(hire_date)        AS hire_day_of_month,
  DAYNAME(hire_date)           AS hire_day_name,
  DAYOFWEEK(hire_date)         AS hire_day_of_week,
  DAYOFYEAR(hire_date)         AS hire_day_of_year
FROM employees;
```

---

## 22. Aggregate Functions

```sql
SELECT
  COUNT(*)                 AS total_employees,
  COUNT(city)               AS employees_with_city,
  COUNT(DISTINCT department) AS distinct_departments,
  MAX(salary)                AS highest_salary,
  MIN(salary)                AS lowest_salary,
  AVG(salary)                AS average_salary,
  SUM(salary)                AS total_payroll,
  GROUP_CONCAT(name)         AS all_names
FROM employees;
```

---

## 23. UPDATE Records

```sql
UPDATE employees
SET bonus = 500
WHERE department = 'Sales';
```

---

## 24. DELETE Records

```sql
DELETE FROM employees
WHERE salary < 28000;
```

---

## 25. Constraints

```sql
-- Already declared above, exercised here:
-- PRIMARY KEY  -> employees.id, departments.dept_id, projects.project_id
-- FOREIGN KEY  -> employees.dept_id references departments.dept_id
-- UNIQUE       -> departments.dept_name
-- NOT NULL     -> employees.name, departments.dept_name
-- DEFAULT      -> employees.status defaults to 'active'
-- CHECK        -> projects.budget must be >= 0
-- CASCADE      -> ON DELETE CASCADE on employees.dept_id
```

Run this on its own — it's meant to fail, demonstrating the `CHECK` constraint:

```sql
INSERT INTO projects (project_name, emp_id, budget) VALUES ('Bad Budget', 1, -100);
```

Run this separately to see `ON DELETE CASCADE` in action (deleting a department removes its employees too):

```sql
SELECT * FROM employees WHERE dept_id = 3;   -- before: Liam Ortega is here
DELETE FROM departments WHERE dept_id = 3;
SELECT * FROM employees WHERE dept_id = 3;   -- after: gone, cascaded automatically
```

---

## 26. Views & Indexes

```sql
CREATE VIEW high_earners AS
SELECT name, department, salary
FROM employees
WHERE salary > 50000;

SELECT * FROM high_earners;

CREATE INDEX idx_employees_city ON employees(city);
```

---

## 27. Schema Introspection & Admin Commands

```sql
SHOW DATABASES;
SHOW TABLES;
SHOW COLUMNS FROM employees;

DESC employees;
DESCRIBE departments;
```

---

## 28. RENAME TABLE

```sql
RENAME TABLE projects TO assignments;
SHOW TABLES;
RENAME TABLE assignments TO projects;
```

---

## 29. TRUNCATE

```sql
-- Function form: numeric truncation
SELECT TRUNCATE(99.999, 1);

-- Statement form: empties a table (rewritten to DELETE FROM under the hood)
TRUNCATE TABLE projects;
```

---

## 30. DUAL

```sql
SELECT 1 + 1 FROM DUAL;
SELECT NOW() FROM DUAL;
```

---

## 31. Cleanup

### Drop Table (with IF EXISTS)

```sql
DROP VIEW IF EXISTS high_earners;   -- this was a VIEW, not a TABLE — DROP TABLE would fail even with IF EXISTS
DROP TABLE IF EXISTS projects;
DROP TABLE IF EXISTS employees;
DROP TABLE IF EXISTS departments;
```

### Drop Database

```sql
DROP DATABASE test_db;
```
