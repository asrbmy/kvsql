# SQL Practice Script

## 1. Database & Table Setup

```sql
CREATE DATABASE test_db;

CREATE TABLE employees (
  id INT PRIMARY KEY,
  name VARCHAR(50) NOT NULL,
  department VARCHAR(30),
  city VARCHAR(30),
  salary INT,
  hire_date DATE
);

ALTER TABLE employees ADD COLUMN bonus INT;
```

---

## 2. INSERT Data

> One record intentionally contains a `NULL` city to demonstrate `IS NULL`.

```sql
INSERT INTO employees (id, name, department, city, salary, hire_date) VALUES
  (1, 'Ariana Cole',   'Sales',       'Austin',  52000, '2023-01-15'),
  (2, 'Devon Marsh',   'Engineering', 'Denver',  91000, '2022-06-21'),
  (3, 'Priya Nair',    'Sales',       NULL,      38000, '2024-03-02'),
  (4, 'Liam Ortega',   'Support',     'Boise',   27000, '2021-11-09'),
  (5, 'Nadia Farrow',  'Engineering', 'Denver', 105000, '2020-08-30');
```

---

## 3. SELECT / FROM / WHERE

```sql
SELECT * FROM employees
WHERE salary > 50000;
```

---

## 4. BETWEEN Operator

```sql
SELECT name, salary
FROM employees
WHERE salary BETWEEN 30000 AND 90000;
```

---

## 5. Logical Operators

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

## 6. IS NULL / IS NOT NULL

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

## 7. UPDATE Records

```sql
UPDATE employees
SET bonus = 500
WHERE department = 'Sales';
```

---

## 8. DELETE Records

```sql
DELETE FROM employees
WHERE salary < 28000;
```

---

## 9. Mathematical Functions

```sql
SELECT
  POWER(2, 5) AS power_result,
  ROUND(91234.5678, 2) AS rounded,
  MOD(17, 5) AS mod_result;
```

---

## 10. String Functions

```sql
SELECT
  UCASE(name)       AS upper_name,
  LCASE(department) AS lower_dept,
  MID(name, 1, 3)   AS mid_test,
  SUBSTRING(name, 3) AS substring_test,
  LENGTH(name)      AS name_length,
  LEFT(name, 4)     AS left_chars,
  RIGHT(name, 4)    AS right_chars,
  INSTR(name, 'a')  AS position_of_a,
  TRIM('  padded  ') AS trimmed,
  LTRIM('  left')   AS left_trimmed,
  RTRIM('right  ')  AS right_trimmed
FROM employees;
```

---

## 11. Date Functions

```sql
SELECT
  NOW() AS current_datetime,
  DATE(hire_date) AS date_only,
  MONTH(hire_date) AS hire_month,
  MONTHNAME(hire_date) AS hire_month_name,
  YEAR(hire_date) AS hire_year,
  DAY(hire_date) AS hire_day,
  DAYNAME(hire_date) AS hire_day_name
FROM employees;
```

---

## 12. Aggregate Functions

```sql
SELECT
  COUNT(*) AS total_employees,
  COUNT(city) AS employees_with_city,
  MAX(salary) AS highest_salary,
  MIN(salary) AS lowest_salary,
  AVG(salary) AS average_salary,
  SUM(salary) AS total_payroll
FROM employees;
```

---

## 13. Describe the Table

```sql
DESC employees;
```

---

## 14. Cleanup

### Drop Table

```sql
DROP TABLE employees;
```

### Drop Database

```sql
DROP DATABASE test_db;
```
