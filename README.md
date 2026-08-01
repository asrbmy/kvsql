# KVSQL — SQL Server Online Editor

A browser-based SQL editor and playground. Write, run, and share SQL — no server, no signup, no install. Everything runs locally in your browser tab.

- **Live demo:** https://asrbmy.github.io/kvsql/
- **Source code:** https://github.com/asrbmy/kvsql
- **Developed by:** ASRBMY

---

## What is this?

KVSQL is a single-page, self-contained SQL editor. You get a code editor with syntax highlighting and autocomplete, a real SQL engine running entirely inside your browser (no data ever leaves your machine), a results grid with sorting/filtering/export, a live schema browser, and a query history — all in one HTML file.

It's built to feel familiar to anyone coming from **MySQL**, while actually running on **SQLite** under the hood. That combination means you get SQLite's speed and simplicity, plus compatibility shims for the MySQL-style commands and functions people usually reach for first (`SHOW TABLES`, `DESCRIBE`, `NOW()`, `CONCAT()`, and more — see the full reference below).

## How it works

| Piece | What it does |
|---|---|
| **[sql.js](https://github.com/sql-js/sql.js)** | SQLite compiled to WebAssembly. This is the actual database engine — it runs in your browser tab, in memory. |
| **[CodeMirror](https://codemirror.net/)** | The code editor: SQL syntax highlighting, line numbers, bracket matching, and `Ctrl/Cmd + Space` autocomplete. |
| **Command shim layer** | Before your SQL reaches the engine, a small preprocessing step rewrites the handful of MySQL-only commands (`SHOW TABLES`, `DESC`, `TRUNCATE`, `CREATE DATABASE`, ...) into SQLite-native equivalents, and registers extra functions (`CONCAT()`, `NOW()`, `MOD()`, ...) that SQLite doesn't ship with. |
| **Results grid** | Renders query output as a sortable, filterable table. Multiple statements in one run get their own tabs. |
| **Schema panel** | Reads `sqlite_master` and `PRAGMA table_info` live, so every table, column, type, and key badge (PK / UNIQUE / NOT NULL) updates automatically after any DDL statement. |

Because the database lives only in your browser tab's memory:
- Nothing is uploaded anywhere — it's 100% client-side.
- Refreshing the page or closing the tab clears your data, unless you explicitly **Save DB** first.
- You can **Open DB** to reload a `.sqlite` file, **Import CSV** to spin up a table from a spreadsheet, and **Save/Open .sql** to move query files in and out.

## Toolbar reference

| Button | Does |
|---|---|
| **Run** (`Ctrl/Cmd + Enter`) | Executes everything in the editor. Multiple statements (separated by `;`) run in one go, each producing its own result tab. |
| **Format** | Tidies whitespace and breaks long queries onto multiple lines at `SELECT`, `FROM`, `WHERE`, `JOIN`, etc. |
| **Explain** | Runs `EXPLAIN QUERY PLAN` on the first statement so you can see how SQLite intends to execute it. |
| **Open .sql / Save .sql** | Load or download the editor's contents as a `.sql` text file. |
| **Open DB / Save DB** | Load or download the entire working database as a portable `.sqlite` file. |
| **Import CSV** | Turns a CSV file into a new table (all columns as `TEXT`), then previews it. |
| **Export CSV** | Downloads the currently visible result set as a CSV file. |
| **Reset** | Drops everything and starts from a completely blank database. |

Destructive statements (`DROP`, `TRUNCATE`, or `DELETE`/`UPDATE` with no `WHERE`) always trigger a confirmation prompt before running.

---

## Supported SQL commands

Everything below runs directly in the editor. Commands are grouped by what's **native to SQLite** (works out of the box) and what's a **MySQL-style shim** built specifically for this editor.

### Database & table structure

| Command | Notes | Example |
|---|---|---|
| `CREATE DATABASE` | Shimmed. This editor has one active in-memory database, so `CREATE DATABASE` just names/acknowledges it rather than spinning up a second database. | `CREATE DATABASE shop;` |
| `DROP DATABASE` | Shimmed. Drops every table in the current workspace. | `DROP DATABASE shop;` |
| `USE` | Shimmed. Sets the "active database" name shown in status messages. | `USE shop;` |
| `CREATE TABLE` | Native. Supports `PRIMARY KEY`, `FOREIGN KEY`, `UNIQUE`, `CHECK`, `DEFAULT`, `NOT NULL`. Foreign keys are enforced (`PRAGMA foreign_keys = ON` is set automatically). | `CREATE TABLE customers (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT NOT NULL, city TEXT);` |
| `AUTO_INCREMENT` | Shimmed → rewritten to SQLite's `AUTOINCREMENT`. Must directly follow `INTEGER PRIMARY KEY`. | `id INTEGER PRIMARY KEY AUTO_INCREMENT` |
| `ALTER TABLE` | Native — add/rename columns, rename table. | `ALTER TABLE customers ADD COLUMN email TEXT;` |
| `DROP TABLE` | Native. | `DROP TABLE customers;` |
| `TRUNCATE [TABLE]` | Shimmed → rewritten to `DELETE FROM table`. | `TRUNCATE TABLE customers;` |
| `RENAME TABLE` | Shimmed → rewritten to SQLite's `ALTER TABLE ... RENAME TO ...`. Supports comma-separated multi-renames. | `RENAME TABLE customers TO clients;` |
| `CREATE VIEW` | Native. | `CREATE VIEW active_customers AS SELECT * FROM customers WHERE city IS NOT NULL;` |
| `CREATE INDEX` | Native. | `CREATE INDEX idx_city ON customers(city);` |
| `DESC` / `DESCRIBE` | Shimmed → rewritten into a `SELECT` over `pragma_table_info()`. Prints Field / Type / Null / Key / Default. | `DESCRIBE customers;` |
| `SHOW TABLES` | Shimmed. | `SHOW TABLES;` |
| `SHOW DATABASES` | Shimmed. | `SHOW DATABASES;` |
| `SHOW COLUMNS FROM table` | Shimmed. Same output as `DESCRIBE`. | `SHOW COLUMNS FROM customers;` |

### Reading and writing data

| Command | Example |
|---|---|
| `SELECT ... FROM ... WHERE` | `SELECT name, city FROM customers WHERE city = 'Austin';` |
| Relational operators `= != <> < > <= >=` | `SELECT * FROM customers WHERE id >= 3;` |
| `BETWEEN` | `SELECT * FROM customers WHERE id BETWEEN 2 AND 5;` |
| `AND` / `OR` / `NOT` | `SELECT * FROM customers WHERE city = 'Austin' AND id > 1;` |
| `IS NULL` / `IS NOT NULL` | `SELECT * FROM customers WHERE city IS NULL;` |
| `IN` / `NOT IN` | `SELECT * FROM customers WHERE city IN ('Austin', 'Denver');` |
| `LIKE` | `SELECT * FROM customers WHERE name LIKE 'A%';` |
| `DISTINCT` | `SELECT DISTINCT city FROM customers;` |
| `ORDER BY` | `SELECT * FROM customers ORDER BY name DESC;` |
| `LIMIT` / `OFFSET` | `SELECT * FROM customers LIMIT 10 OFFSET 5;` |
| `GROUP BY` / `HAVING` | `SELECT city, COUNT(*) FROM customers GROUP BY city HAVING COUNT(*) > 1;` |
| `JOIN` (`INNER` / `LEFT` / `RIGHT` / `CROSS`) | `SELECT o.id, c.name FROM orders o JOIN customers c ON o.customer_id = c.id;` |
| `UNION` / `UNION ALL` | `SELECT city FROM customers UNION SELECT city FROM suppliers;` |
| Subqueries | `SELECT * FROM customers WHERE id IN (SELECT customer_id FROM orders);` |
| `CASE WHEN` | `SELECT name, CASE WHEN city IS NULL THEN 'Unknown' ELSE city END AS city FROM customers;` |
| `WITH` (CTE) | `WITH big_cities AS (SELECT city FROM customers GROUP BY city HAVING COUNT(*) > 2) SELECT * FROM big_cities;` |
| `INSERT` | `INSERT INTO customers (name, city) VALUES ('Ariana Cole', 'Austin');` |
| `UPDATE` | `UPDATE customers SET city = 'Boise' WHERE id = 1;` |
| `DELETE` | `DELETE FROM customers WHERE id = 1;` |

### Math functions

| Function | Example |
|---|---|
| `POWER(x, y)` | `SELECT POWER(2, 5);` → `32` |
| `MOD(x, y)` | `SELECT MOD(11, 5);` → `1` |
| `ROUND(x, d)` | `SELECT ROUND(15.789, 2);` → `15.79` |
| `ABS(x)` | `SELECT ABS(-8);` → `8` |
| `CEIL(x)` / `CEILING(x)` | `SELECT CEIL(4.2);` → `5` |
| `FLOOR(x)` | `SELECT FLOOR(4.8);` → `4` |
| `SIGN(x)` | `SELECT SIGN(-20);` → `-1` |
| `SQRT(x)` | `SELECT SQRT(144);` → `12` |
| `TRUNCATE(x, d)` | Shimmed. Truncates toward zero without rounding — different from the `TRUNCATE TABLE` statement above. `SELECT TRUNCATE(15.789, 2);` → `15.78` |

### String functions

| Function | Example |
|---|---|
| `CHAR(code)` | Native — builds a string from character codes. `SELECT CHAR(65);` → `A` |
| `UCASE(s)` / `UPPER(s)` | `SELECT UCASE('austin');` → `AUSTIN` |
| `LCASE(s)` / `LOWER(s)` | `SELECT LCASE('AUSTIN');` → `austin` |
| `MID(s, start, len)` / `SUBSTRING(s, start, len)` / `SUBSTR(s, start, len)` | `SELECT MID('database', 1, 4);` → `data` |
| `LENGTH(s)` | `SELECT LENGTH('sql');` → `3` |
| `LEFT(s, n)` | `SELECT LEFT('database', 4);` → `data` |
| `RIGHT(s, n)` | `SELECT RIGHT('database', 4);` → `base` |
| `INSTR(s, sub)` | `SELECT INSTR('database', 'a');` → `2` |
| `LTRIM(s)` / `RTRIM(s)` / `TRIM(s)` | `SELECT TRIM('  hi  ');` → `hi` |
| `CONCAT(a, b, ...)` | `SELECT CONCAT(name, ' — ', city) FROM customers;` |
| `REPLACE(s, find, repl)` | `SELECT REPLACE('2024-01-01', '-', '/');` → `2024/01/01` |
| `COALESCE(a, b, ...)` / `IFNULL(a, b)` | `SELECT COALESCE(city, 'Unknown') FROM customers;` |
| `CAST(x AS type)` | `SELECT CAST('42' AS INTEGER);` |

### Date & time functions

| Function | Example |
|---|---|
| `NOW()` | `SELECT NOW();` → current UTC date/time, e.g. `2026-08-01 09:14:02` |
| `SYSDATE()` | Shimmed, same as `NOW()`. `SELECT SYSDATE();` |
| `CURDATE()` | Shimmed. `SELECT CURDATE();` → current UTC date, e.g. `2026-08-01` |
| `CURRENT_DATE()` | Shimmed — SQLite's `CURRENT_DATE` is a bare keyword, so `CURRENT_DATE()` is rewritten to drop the parentheses automatically. `SELECT CURRENT_DATE();` |
| `DATE(x)` | Native. `SELECT DATE('2024-05-20 10:30:00');` → `2024-05-20` |
| `MONTH(x)` | `SELECT MONTH('2024-05-20');` → `5` |
| `MONTHNAME(x)` | `SELECT MONTHNAME('2024-05-20');` → `May` |
| `YEAR(x)` | `SELECT YEAR('2024-05-20');` → `2024` |
| `DAY(x)` / `DAYOFMONTH(x)` | `SELECT DAYOFMONTH('2024-05-20');` → `20` |
| `DAYNAME(x)` | `SELECT DAYNAME('2024-05-20');` → `Monday` |
| `DAYOFWEEK(x)` | Shimmed, MySQL numbering (1 = Sunday ... 7 = Saturday). `SELECT DAYOFWEEK('2024-05-20');` → `2` |
| `DAYOFYEAR(x)` | Shimmed. `SELECT DAYOFYEAR('2024-05-20');` → `141` |

### Joins & table aliases

All native — no shim required.

| Join type | Example |
|---|---|
| `CROSS JOIN` | `SELECT * FROM table1 CROSS JOIN table2;` |
| Cartesian product (implicit) | `SELECT * FROM table1, table2;` |
| Equi join (implicit, via `WHERE`) | `SELECT * FROM emp, dept WHERE emp.deptno = dept.deptno;` |
| `INNER JOIN` / `JOIN` | `SELECT * FROM emp JOIN dept ON emp.deptno = dept.deptno;` |
| `NATURAL JOIN` | `SELECT * FROM emp NATURAL JOIN dept;` |
| `LEFT JOIN` | `SELECT * FROM emp LEFT JOIN project ON emp.emp_id = project.emp_id;` |
| `RIGHT JOIN` | `SELECT * FROM emp RIGHT JOIN project ON emp.emp_id = project.emp_id;` |
| `USING` | `SELECT * FROM emp JOIN dept USING (deptno);` |
| Table alias | `SELECT e.name, d.dept_name FROM employees e JOIN departments d ON e.department_id = d.dept_id;` |

### Aggregate functions

| Function | Example |
|---|---|
| `COUNT(*)` / `COUNT(col)` | `SELECT COUNT(*) FROM customers;` |
| `COUNT(DISTINCT col)` | `SELECT COUNT(DISTINCT city) FROM customers;` |
| `MAX(col)` | `SELECT MAX(salary) FROM employees;` |
| `MIN(col)` | `SELECT MIN(salary) FROM employees;` |
| `AVG(col)` | `SELECT AVG(salary) FROM employees;` |
| `SUM(col)` | `SELECT SUM(salary) FROM employees;` |
| `GROUP_CONCAT(col)` | `SELECT GROUP_CONCAT(name) FROM customers;` |

---

## End-to-end example

```sql
CREATE DATABASE shop;

CREATE TABLE customers (
  id INTEGER PRIMARY KEY AUTO_INCREMENT,
  name TEXT NOT NULL,
  city TEXT,
  signed_up_on DATE
);

INSERT INTO customers (name, city, signed_up_on) VALUES
  ('Ariana Cole', 'Austin', '2024-01-15'),
  ('Devon Marsh', 'Denver', '2024-03-02'),
  ('Priya Nair', NULL, '2024-06-21');

SELECT
  UCASE(name)          AS name_upper,
  CONCAT(name, ' — ', COALESCE(city, 'Unknown')) AS summary,
  MONTHNAME(signed_up_on) AS signup_month
FROM customers
WHERE signed_up_on BETWEEN '2024-01-01' AND '2024-12-31'
ORDER BY signed_up_on DESC;

SHOW TABLES;
DESCRIBE customers;
```

---

## Running it locally

This is a single static HTML file with no build step:

1. Download `index.html` (or clone the repo: `git clone https://github.com/asrbmy/kvsql.git`).
2. Open it directly in a browser, or serve it with any static file server (e.g. `npx serve .`).
3. That's it — the SQLite WASM engine and editor libraries load from a CDN, and the database itself runs entirely in your tab.

Or just use the hosted version: **https://asrbmy.github.io/kvsql/**

## Data types

SQLite doesn't enforce strict column types the way MySQL does — it accepts **any** type name in a column definition and infers a storage affinity from it (e.g. a name containing `INT` gets integer affinity, `CHAR`/`TEXT`/`CLOB` gets text affinity, `REAL`/`FLOA`/`DOUB` gets real affinity, otherwise numeric affinity). Practically, that means all of these just work as column types with no changes needed:

- **Numeric:** `INT`, `TINYINT`, `SMALLINT`, `MEDIUMINT`, `BIGINT`, `FLOAT`, `DOUBLE`, `DECIMAL`
- **Character:** `CHAR`, `VARCHAR`, `TEXT`, `BLOB`
- **Date & time:** `DATE`, `TIME`, `DATETIME`, `TIMESTAMP`, `YEAR`

One exception: **`ENUM('a', 'b', 'c')`** isn't valid SQLite syntax (its type grammar doesn't accept a quoted value list in parentheses). It's shimmed — rewritten to `TEXT` plus a matching `CHECK` constraint:

```sql
CREATE TABLE orders (
  id INTEGER PRIMARY KEY,
  status ENUM('pending', 'shipped', 'delivered') DEFAULT 'pending'
);
```
becomes, under the hood:
```sql
CREATE TABLE orders (
  id INTEGER PRIMARY KEY,
  status TEXT CHECK(status IN ('pending', 'shipped', 'delivered')) DEFAULT 'pending'
);
```
Querying and inserting into `orders` works exactly as you'd expect — invalid values are rejected by the `CHECK` constraint the same way MySQL would reject an out-of-range `ENUM` value.

## SQL objects & vocabulary

`DATABASE`, `TABLE`, `COLUMN`, `ROW`, `RECORD`, `FIELD`, `VIEW`, `INDEX`, and `CONSTRAINT` aren't standalone commands — they're the nouns used throughout the DDL above (`CREATE TABLE`, `ALTER TABLE ... ADD COLUMN`, `CREATE VIEW`, `CREATE INDEX`, `PRIMARY KEY`/`FOREIGN KEY`/`CHECK` constraints, etc.), all covered in the sections above.

## Limitations

- One active database per session — `CREATE DATABASE` doesn't create a genuinely separate, isolated database.
- Data is in-memory only unless you explicitly **Save DB**; refreshing the page clears it.
- `SOURCE file.sql;` (MySQL client command to run a script off the server's filesystem) has no browser equivalent — use the **Open .sql** button instead. Typing `SOURCE ...` shows a reminder rather than erroring.
- `FROM DUAL` is accepted and silently dropped (e.g. `SELECT 1 FROM DUAL;` just runs as `SELECT 1;`), since SQLite doesn't need a dummy table for expression-only queries.
- Stored procedures, triggers-as-MySQL-syntax, and MySQL-specific storage engine options aren't supported (SQLite doesn't have an equivalent concept).

## Credits

Developed by **ASRBMY**.
- GitHub: https://github.com/asrbmy/kvsql
- Live demo: https://asrbmy.github.io/kvsql/
