# PostgreSQL Learning Notes
*Date: April 9, 2026*

---

## Table of Contents
1. [UPDATE SET FROM WHERE Syntax](#1-update-set-from-where-syntax)
2. [PostgreSQL vs Other Databases – Handling Updates with Joins](#2-postgresql-vs-other-databases--handling-updates-with-joins)
3. [CTEs (Common Table Expressions)](#3-ctes-common-table-expressions)
4. [Eliminating Duplicates in CTEs](#4-eliminating-duplicates-in-ctes)
5. [Window Functions & OVER PARTITION BY](#5-window-functions--over-partition-by)
6. [Filtering Inside a CTE](#6-filtering-inside-a-cte)
7. [Practice Exercises – CTEs & Window Functions](#7-practice-exercises--ctes--window-functions)
8. [Setting Up PostgreSQL Locally](#8-setting-up-postgresql-locally)

---

## 1. UPDATE SET FROM WHERE Syntax

### What is it?
PostgreSQL supports a special `UPDATE ... SET ... FROM ... WHERE` syntax that allows you to **join another table** (or subquery/CTE) during an update.

### Basic Structure
```sql
UPDATE target_table
SET target_column = source_table.some_column
FROM source_table
WHERE target_table.join_key = source_table.join_key;
```

### How to Read It
- **`UPDATE target_table`** → the table you want to modify
- **`SET target_column = ...`** → the column and new value you want to assign
- **`FROM source_table`** → an additional table (or CTE/subquery) that provides the new values
- **`WHERE`** → the join condition that links the two tables together

### What is `FROM` doing?
The `FROM` clause **brings in an external data source** to use in the update. Without it, you can only set a column to a literal value or expression based on the row itself. With `FROM`, you can reference values from **another table or query result**.

### Example
```sql
UPDATE dc_case
SET parent_case_id = mapping.ecollect_case_id
FROM mapping
WHERE dc_case.dc_case_id = mapping.dc_case_id;
```
This reads: *"For each row in `dc_case`, find the matching row in `mapping` where the `dc_case_id` matches, and set `parent_case_id` to the value of `ecollect_case_id` from that matching row."*

---

## 2. PostgreSQL vs Other Databases – Handling Updates with Joins

### PostgreSQL
Uses `UPDATE ... FROM`:
```sql
UPDATE orders
SET status = c.preferred_status
FROM customers c
WHERE orders.customer_id = c.id;
```

### MySQL / MariaDB
Uses `UPDATE ... JOIN` directly:
```sql
UPDATE orders o
JOIN customers c ON o.customer_id = c.id
SET o.status = c.preferred_status;
```

### SQL Server (T-SQL)
Similar to PostgreSQL but slightly different:
```sql
UPDATE o
SET o.status = c.preferred_status
FROM orders o
JOIN customers c ON o.customer_id = c.id;
```

### Oracle
Uses a **correlated subquery** or `MERGE`:
```sql
UPDATE orders o
SET o.status = (
    SELECT c.preferred_status
    FROM customers c
    WHERE c.id = o.customer_id
);
```

### Key Differences Summary

| Feature                        | PostgreSQL         | MySQL              | SQL Server         | Oracle             |
|-------------------------------|--------------------|--------------------|--------------------|--------------------|
| Syntax                        | `UPDATE ... FROM`  | `UPDATE ... JOIN`  | `UPDATE ... FROM`  | Correlated subquery or `MERGE` |
| CTE in UPDATE                 | ✅ Yes             | ❌ No              | ✅ Yes             | ✅ Yes (Oracle 12c+)|
| Multiple table update         | ❌ No (one target) | ✅ Yes             | ❌ No (one target) | ❌ No              |

---

## 3. CTEs (Common Table Expressions)

### What is a CTE?
A CTE (Common Table Expression) is a **named temporary result set** that you define at the beginning of a query using the `WITH` keyword. It is only available within the scope of that query.

### Syntax
```sql
WITH cte_name AS (
    SELECT ...
    FROM ...
    WHERE ...
)
SELECT * FROM cte_name;
```

### Why Use CTEs?
- Makes complex queries more **readable and modular**
- Allows you to **reference the same subquery multiple times**
- Enables **recursive queries**
- Can be used with `UPDATE`, `DELETE`, and `INSERT`

### Example: CTE used in an UPDATE
```sql
WITH mapping AS (
    SELECT e.ecollect_case_id, d.dc_case_id
    FROM ecollect_cases e
    JOIN dc_cases d ON e.some_key = d.some_key
)
UPDATE dc_case
SET parent_case_id = mapping.ecollect_case_id
FROM mapping
WHERE dc_case.dc_case_id = mapping.dc_case_id;
```

---

## 4. Eliminating Duplicates in CTEs

### The Problem
Sometimes a CTE may contain rows where one `ecollect_case_id` is linked to **multiple** `dc_case_id` values. You want to **completely exclude** all rows related to such `ecollect_case_id`s — not even keeping one.

### Option 1: Using a Window Function (`COUNT OVER PARTITION BY`)

```sql
WITH raw_mapping AS (
    SELECT e.ecollect_case_id, d.dc_case_id
    FROM ecollect_cases e
    JOIN dc_cases d ON e.some_key = d.some_key
),
mapping AS (
    SELECT
        ecollect_case_id,
        dc_case_id,
        COUNT(*) OVER (PARTITION BY ecollect_case_id) AS match_count
    FROM raw_mapping
)
SELECT ecollect_case_id, dc_case_id
FROM mapping
WHERE match_count = 1;
```

### Option 2: Using `GROUP BY` + `HAVING`

```sql
WITH mapping AS (
    SELECT e.ecollect_case_id, d.dc_case_id
    FROM ecollect_cases e
    JOIN dc_cases d ON e.some_key = d.some_key
    GROUP BY e.ecollect_case_id, d.dc_case_id
    HAVING COUNT(*) = 1
),
...
```

> **Note:** The `HAVING COUNT(*) = 1` approach works at the pair level, but to truly exclude `ecollect_case_id`s that appear with **multiple** `dc_case_id`s, the window function approach is more precise.

---

## 5. Window Functions & OVER PARTITION BY

### What is a Window Function?
A **window function** performs a calculation across a **set of rows related to the current row** — without collapsing those rows (unlike `GROUP BY` which reduces rows).

The result is added as a **new column** to the existing rows, with the **same value repeated** for all rows that belong to the same group (partition).

### General Syntax
```sql
<aggregate_function>(<column>) OVER (PARTITION BY <grouping_column> ORDER BY <ordering_column>)
```

- **`PARTITION BY`** → defines the groups (like `GROUP BY`, but rows are not collapsed)
- **`ORDER BY`** (optional) → defines the order within each partition (used by ranking/cumulative functions)

### Common Window Functions

| Function         | Description                                      |
|-----------------|--------------------------------------------------|
| `COUNT(*)`       | Count rows in the partition                      |
| `SUM(col)`       | Sum values in the partition                      |
| `AVG(col)`       | Average of values in the partition               |
| `MIN(col)`       | Minimum value in the partition                   |
| `MAX(col)`       | Maximum value in the partition                   |
| `ROW_NUMBER()`   | Assigns a unique sequential number per partition |
| `RANK()`         | Rank with gaps for ties                          |
| `DENSE_RANK()`   | Rank without gaps for ties                       |
| `LAG(col)`       | Value from the previous row                      |
| `LEAD(col)`      | Value from the next row                          |

### Example
```sql
SELECT
    department,
    employee_name,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS avg_dept_salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees;
```

**Output (example):**

| department | employee_name | salary | avg_dept_salary | salary_rank |
|------------|---------------|--------|-----------------|-------------|
| HR         | Alice         | 5000   | 4500            | 1           |
| HR         | Bob           | 4000   | 4500            | 2           |
| IT         | Carol         | 7000   | 6500            | 1           |
| IT         | Dave          | 6000   | 6500            | 2           |

> The rows are **not collapsed** — every row keeps its identity, but gains context from its group.

### Key Insight
> `OVER PARTITION BY` is for **aggregating without changing the structure** — it just shows the aggregation result in a new column that will have the **same value repeated** over all rows belonging to the same group.

---

## 6. Filtering Inside a CTE

### Can you put `WHERE` inside a CTE to filter window function results?

**Not directly** — you **cannot** use a window function result in a `WHERE` clause of the **same query level** where it's defined. This is because `WHERE` is evaluated **before** window functions in SQL's logical processing order.

### ❌ This does NOT work:
```sql
WITH mapping AS (
    SELECT
        ecollect_case_id,
        dc_case_id,
        COUNT(*) OVER (PARTITION BY ecollect_case_id) AS match_count
    FROM raw_mapping
    WHERE match_count = 1  -- ❌ match_count not yet available here
)
```

### ✅ Correct approach — wrap it in another CTE or subquery:
```sql
WITH raw_mapping AS (
    SELECT ecollect_case_id, dc_case_id
    FROM ...
),
mapping_with_count AS (
    SELECT
        ecollect_case_id,
        dc_case_id,
        COUNT(*) OVER (PARTITION BY ecollect_case_id) AS match_count
    FROM raw_mapping
),
clean_mapping AS (
    SELECT ecollect_case_id, dc_case_id
    FROM mapping_with_count
    WHERE match_count = 1  -- ✅ Now it works
)
SELECT * FROM clean_mapping;
```

### SQL Logical Processing Order
```
FROM → WHERE → GROUP BY → HAVING → SELECT → WINDOW FUNCTIONS → ORDER BY
```
This is why window function results can only be filtered in an **outer query or subsequent CTE**.

---

## 7. Practice Exercises – CTEs & Window Functions

### Setup: Sample Schema

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    department VARCHAR(50),
    hire_date DATE,
    salary NUMERIC(10, 2),
    manager_id INT
);

INSERT INTO employees (name, department, hire_date, salary, manager_id) VALUES
('Alice',   'HR',      '2018-03-01', 5000, NULL),
('Bob',     'HR',      '2020-07-15', 4000, 1),
('Carol',   'IT',      '2017-01-10', 7000, NULL),
('Dave',    'IT',      '2019-05-20', 6000, 3),
('Eve',     'IT',      '2021-11-01', 5500, 3),
('Frank',   'Finance', '2016-08-25', 8000, NULL),
('Grace',   'Finance', '2022-02-14', 4500, 6),
('Heidi',   'Finance', '2023-06-30', 4200, 6),
('Ivan',    'HR',      '2015-09-09', 6000, NULL),
('Judy',    'IT',      '2022-12-01', 5000, 3);
```

---

### Exercise 1 – Basic CTE: Employees earning above department average

**Goal:** List all employees whose salary is above the average salary of their department.

```sql
WITH dept_avg AS (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT e.name, e.department, e.salary, d.avg_salary
FROM employees e
JOIN dept_avg d ON e.department = d.department
WHERE e.salary > d.avg_salary
ORDER BY e.department, e.salary DESC;
```

---

### Exercise 2 – Window Function: Rank employees by salary within department

**Goal:** Rank employees by salary (highest first) within each department.

```sql
SELECT
    name,
    department,
    salary,
    RANK() OVER (PARTITION BY department ORDER BY salary DESC) AS salary_rank
FROM employees
ORDER BY department, salary_rank;
```

> Try also with `DENSE_RANK()` and `ROW_NUMBER()` — notice the difference when salaries are tied.

---

### Exercise 3 – COUNT OVER: Find departments with more than 2 employees

**Goal:** Using a window function, tag each employee with their department's headcount, then filter.

```sql
WITH tagged AS (
    SELECT
        name,
        department,
        COUNT(*) OVER (PARTITION BY department) AS dept_headcount
    FROM employees
)
SELECT *
FROM tagged
WHERE dept_headcount > 2
ORDER BY department;
```

---

### Exercise 4 – Running Total: Cumulative salary per department ordered by hire date

**Goal:** Show a running total of salaries within each department, ordered by hire date.

```sql
SELECT
    name,
    department,
    hire_date,
    salary,
    SUM(salary) OVER (
        PARTITION BY department
        ORDER BY hire_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_salary
FROM employees
ORDER BY department, hire_date;
```

---

### Exercise 5 – LAG/LEAD: Compare each employee's salary to the next hire in their department

**Goal:** Show for each employee what the next hired person in the same department earns.

```sql
SELECT
    name,
    department,
    hire_date,
    salary,
    LEAD(salary) OVER (PARTITION BY department ORDER BY hire_date) AS next_hire_salary,
    LEAD(name)   OVER (PARTITION BY department ORDER BY hire_date) AS next_hire_name
FROM employees
ORDER BY department, hire_date;
```

---

### Exercise 6 – Eliminate duplicates with window function (mirrors the real use case)

**Goal:** Suppose a mapping table links `ecollect_id` to `dc_id`, but some `ecollect_id`s appear multiple times. Remove all ambiguous ones.

```sql
CREATE TABLE case_mapping (
    ecollect_id INT,
    dc_id INT
);

INSERT INTO case_mapping VALUES
(101, 1), (101, 2),  -- ambiguous: ecollect_id 101 maps to 2 dc_ids
(102, 3),            -- unique
(103, 4), (103, 5),  -- ambiguous
(104, 6);            -- unique

WITH counted AS (
    SELECT
        ecollect_id,
        dc_id,
        COUNT(*) OVER (PARTITION BY ecollect_id) AS cnt
    FROM case_mapping
)
SELECT ecollect_id, dc_id
FROM counted
WHERE cnt = 1;
-- Expected result: only (102, 3) and (104, 6)
```

---

### Exercise 7 – Chained CTEs: Top earner per department with their manager

**Goal:** Find the top earner in each department, then look up their manager's name.

```sql
WITH ranked AS (
    SELECT
        id,
        name,
        department,
        salary,
        manager_id,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
),
top_earners AS (
    SELECT * FROM ranked WHERE rn = 1
)
SELECT
    t.name AS top_earner,
    t.department,
    t.salary,
    m.name AS manager_name
FROM top_earners t
LEFT JOIN employees m ON t.manager_id = m.id
ORDER BY t.department;
```

---

## 8. Setting Up PostgreSQL Locally

### Prerequisites Checklist

| Tool          | Purpose                              | Check Command               |
|---------------|--------------------------------------|-----------------------------|
| PostgreSQL    | The database engine                  | `psql --version`            |
| psql          | CLI client (bundled with PostgreSQL) | `psql --version`            |
| pgAdmin 4     | GUI client (optional but helpful)    | Open from Start Menu        |
| DBeaver       | Alternative GUI client (optional)    | Open from Start Menu        |

### Step 1 – Verify PostgreSQL is installed

```powershell
psql --version
```
If not installed, download from: https://www.postgresql.org/download/windows/

### Step 2 – Start the PostgreSQL service (Windows)

```powershell
# Check status
Get-Service -Name postgresql*

# Start if not running
Start-Service -Name postgresql-x64-15   # replace 15 with your version
```

Or via Services GUI: `Win + R` → `services.msc` → find PostgreSQL → Start

### Step 3 – Connect to PostgreSQL

```powershell
psql -U postgres
```
Enter the password you set during installation.

### Step 4 – Create a practice database

```sql
CREATE DATABASE practice_db;
\c practice_db
```

### Step 5 – Create the practice schema and run exercises

Paste the `CREATE TABLE` and `INSERT` statements from Exercise section above, then run the queries.

### Step 6 – Useful psql commands

| Command            | Description                          |
|--------------------|--------------------------------------|
| `\l`               | List all databases                   |
| `\c dbname`        | Connect to a database                |
| `\dt`              | List all tables                      |
| `\d tablename`     | Describe a table                     |
| `\i file.sql`      | Run a SQL file                       |
| `\q`               | Quit psql                            |
| `\x`               | Toggle expanded output (pretty print)|

### Optional: Use pgAdmin or DBeaver
If you prefer a GUI:
- **pgAdmin 4**: comes bundled with PostgreSQL installer
- **DBeaver**: free download at https://dbeaver.io/ — supports PostgreSQL and many other DBs

---

*End of Notes*

