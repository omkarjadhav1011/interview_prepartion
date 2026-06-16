# 10 — Database & SQL Interview

> **Purpose:** 60-question interview preparation guide for Database & SQL rounds at FAANG and top Indian product companies (Flipkart, Razorpay, Swiggy, Groww, CRED, PhonePe). Targeting SDE-1/SDE-2 roles (15+ LPA). All SQL uses **PostgreSQL dialect**.

## Difficulty Legend
- 🟢 Easy — expected in OA / first phone screen
- 🟡 Medium — standard onsite bar
- 🔴 Hard — senior bar or system-design-adjacent
- ⚫ Expert — staff-level, rare but decisive

## Frequency Legend
- 🔥🔥🔥 Very common — appears in >60% of loops
- 🔥🔥 Common — appears in 30–60% of loops
- 🔥 Occasional — appears in <30% of loops

## Table of Contents

### Part 1: SQL Queries (Q1–Q40)
- Q1: Second highest salary (DENSE_RANK, LIMIT+OFFSET, subquery)
- Q2: Nth highest salary (generalized)
- Q3: Employees earning more than their manager (self-join)
- Q4: Find duplicate emails; delete keeping lowest id
- Q5: Department top 3 salaries (ROW_NUMBER vs DENSE_RANK, tie behavior)
- Q6: Three or more consecutive numbers (self-join + LAG)
- Q7: Consecutive login days — longest streak per user
- Q8: Rising temperature (LAG vs self-join)
- Q9: Tree node classification — Root / Inner / Leaf
- Q10: Rank scores without RANK() — simulate DENSE_RANK
- Q11: Trips and users cancellation rate
- Q12: Exchange adjacent seats
- Q13: Customers who never placed an order (LEFT JOIN, NOT EXISTS, NOT IN)
- Q14: Department highest salary (GROUP BY + JOIN)
- Q15: Employees in every department
- Q16: Running total / cumulative sum (SUM OVER)
- Q17: Year-over-year growth % (LAG with PARTITION BY)
- Q18: 7-day moving average (ROWS BETWEEN)
- Q19: Pivot rows to columns (static CASE pivot)
- Q20: Gap and island detection
- Q21: Median salary per department
- Q22: First and last login per user
- Q23: Retention rate — users active in consecutive months
- Q24: Session reconstruction from raw events
- Q25: Funnel analysis — step-by-step conversion
- Q26: Percentile rank of employees by salary
- Q27: Recursive CTE — org hierarchy
- Q28: Recursive CTE — BFS tree traversal
- Q29: Products bought together (market basket / self-join)
- Q30: Longest session duration per user
- Q31: Rolling 30-day distinct active users
- Q32: Top N per group with ties (all approaches)
- Q33: Date spine — generate continuous date series
- Q34: JSONB extraction and filtering (PostgreSQL)
- Q35: Full-text search with tsvector / tsquery
- Q36: CTE vs subquery vs temp table — when to use each
- Q37: EXPLAIN ANALYZE — reading a query plan
- Q38: Deadlock scenario — reproduce and fix
- Q39: Upsert with ON CONFLICT DO UPDATE
- Q40: Lateral join — correlated subquery as table

### Part 2: Database Concepts (Q41–Q60)
- Q41: ACID properties — real-world examples of each
- Q42: Isolation levels — dirty read, non-repeatable read, phantom read
- Q43: Index types — B-tree, Hash, GIN, BRIN
- Q44: Composite index column order — why it matters
- Q45: Index-only scan vs heap fetch
- Q46: Covering index
- Q47: Normalization — 1NF through BCNF with examples
- Q48: When to denormalize — trade-offs
- Q49: CAP theorem — real system examples
- Q50: Sharding strategies — range, hash, directory
- Q51: Read replicas — replication lag and use cases
- Q52: Connection pooling — PgBouncer modes
- Q53: Vacuum and autovacuum in PostgreSQL
- Q54: MVCC — how PostgreSQL implements it
- Q55: WAL — write-ahead logging and crash recovery
- Q56: Partitioning — range, list, hash in PostgreSQL
- Q57: Foreign key vs application-level integrity — trade-offs
- Q58: Optimistic vs pessimistic locking
- Q59: Event sourcing vs traditional CRUD — when to choose each
- Q60: Database migration strategies — zero-downtime schema changes

---

### Q1: Second Highest Salary
**Company:** Amazon, Flipkart, Razorpay, Infosys, TCS  **Difficulty:** 🟢  **Frequency:** 🔥🔥🔥  **Round:** OA/Phone

**Question:** Write a SQL query to find the second highest salary from an `employees` table. Return `NULL` if there is no second highest salary.

**What interviewer is testing:** Whether you know multiple approaches to ranking problems, handle edge cases (ties, single row), and understand trade-offs between approaches.

**Ideal Answer:**

```sql
-- Setup
CREATE TABLE employees (
    id     SERIAL PRIMARY KEY,
    name   VARCHAR(100),
    salary NUMERIC
);

INSERT INTO employees (name, salary) VALUES
    ('Alice',  90000),
    ('Bob',    75000),
    ('Carol',  75000),
    ('Dave',   60000),
    ('Eve',    90000);

-- ─────────────────────────────────────────────
-- Approach 1: DENSE_RANK  (recommended)
-- ─────────────────────────────────────────────
SELECT salary AS second_highest_salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk = 2
LIMIT 1;

/*
Expected output:
 second_highest_salary
-----------------------
 75000
*/

-- Why DENSE_RANK not RANK?
-- Alice and Eve both earn 90000 → they share rank 1.
-- RANK() would skip rank 2 and give Bob/Carol rank 3.
-- DENSE_RANK() gives Bob/Carol rank 2 — correct second-highest.

-- ─────────────────────────────────────────────
-- Approach 2: LIMIT + OFFSET
-- ─────────────────────────────────────────────
SELECT DISTINCT salary AS second_highest_salary
FROM employees
ORDER BY salary DESC
LIMIT 1 OFFSET 1;

/*
Expected output:
 second_highest_salary
-----------------------
 75000
*/

-- DISTINCT collapses the two 90000 rows into one before OFFSET.
-- Without DISTINCT: OFFSET 1 skips the second row (Eve 90000)
-- and lands on Bob 75000 — coincidentally correct here but wrong
-- if there were three rows at rank 1.

-- ─────────────────────────────────────────────
-- Approach 3: Correlated subquery (handles NULL natively)
-- ─────────────────────────────────────────────
SELECT MAX(salary) AS second_highest_salary
FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);

/*
Expected output:
 second_highest_salary
-----------------------
 75000
*/

-- If the table has only one distinct salary, MAX(salary) WHERE
-- salary < max returns NULL — exactly what the problem requires.
-- This is the cleanest approach for the NULL-return requirement.

-- Edge case test: only one row
INSERT INTO employees (name, salary) VALUES ('Lone', 50000);
-- (Then imagine we only have that one row)
-- Subquery: MAX(salary) WHERE salary < 50000 → NULL ✓
-- OFFSET approach: LIMIT 1 OFFSET 1 → empty result set (not NULL) ✗
-- So wrap OFFSET approach in a subquery to coerce empty → NULL:
SELECT (
    SELECT DISTINCT salary FROM employees ORDER BY salary DESC LIMIT 1 OFFSET 1
) AS second_highest_salary;
```

**Follow-up questions the interviewer will ask:**

1. **"What if there are ties at the top — e.g., three people with the same highest salary?"**
   - With `DENSE_RANK`, all three get rank 1, the next distinct salary gets rank 2 — correct.
   - With `MAX(salary) WHERE salary < MAX(salary)` — also correct because it ignores the top value entirely.
   - With `DISTINCT ... LIMIT 1 OFFSET 1` — also correct because DISTINCT collapses duplicates first.

2. **"How would you generalize this to the Nth highest salary?"**
   - See Q2. Short answer: `DENSE_RANK() = N` or `LIMIT 1 OFFSET N-1` on a `SELECT DISTINCT` subquery.

3. **"Which approach is most performant?"**
   - The correlated subquery does two full scans (one for MAX, one for the filtered MAX). On a large table without an index on `salary`, all approaches are O(n). With a B-tree index on `salary`, the index can be scanned in reverse; the subquery approach can use the index for both MAX calls, making it fastest. The window function approach scans once but materializes a derived table.

**Common mistakes that get you rejected:**
- Using `RANK()` instead of `DENSE_RANK()` — breaks on ties at the top.
- Forgetting `DISTINCT` in the `LIMIT/OFFSET` approach — gives wrong answer with ties.
- Not handling the NULL case when there is no second highest salary.
- Returning an empty result set instead of a NULL row — many judges expect a single row with NULL.

---

### Q2: Nth Highest Salary (Generalized)
**Company:** Google, Microsoft, Amazon, Wipro, Cognizant  **Difficulty:** 🟡  **Frequency:** 🔥🔥🔥  **Round:** OA/Phone

**Question:** Write a SQL query (or function) to find the Nth highest salary from the `employees` table. Return NULL if it does not exist.

**What interviewer is testing:** Parameterization of ranking queries, use of window functions or offset arithmetic, and handling of edge cases.

**Ideal Answer:**

```sql
-- Reuse the employees table from Q1.

-- ─────────────────────────────────────────────
-- Approach 1: DENSE_RANK window function
-- ─────────────────────────────────────────────
-- Replace :n with the desired rank (e.g., 3)
SELECT salary AS nth_highest_salary
FROM (
    SELECT salary,
           DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM employees
) ranked
WHERE rnk = 3   -- change 3 to N
LIMIT 1;

/*
With our data (90000, 75000, 60000, 50000):
N=1 → 90000
N=2 → 75000
N=3 → 60000
N=4 → 50000
N=5 → NULL (no 5th distinct salary)
*/

-- ─────────────────────────────────────────────
-- Approach 2: DISTINCT + OFFSET (N-1)
-- ─────────────────────────────────────────────
SELECT (
    SELECT DISTINCT salary
    FROM employees
    ORDER BY salary DESC
    LIMIT 1 OFFSET 2   -- OFFSET = N - 1, so N=3 → OFFSET 2
) AS nth_highest_salary;

-- ─────────────────────────────────────────────
-- PostgreSQL CREATE FUNCTION wrapper (for reuse)
-- ─────────────────────────────────────────────
CREATE OR REPLACE FUNCTION nth_highest_salary(n INT)
RETURNS NUMERIC AS $$
    SELECT salary
    FROM (
        SELECT salary,
               DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
        FROM employees
    ) r
    WHERE rnk = n
    LIMIT 1;
$$ LANGUAGE sql STABLE;

SELECT nth_highest_salary(3);
-- Returns: 60000
```

**Follow-up questions the interviewer will ask:**

1. **"Why use DENSE_RANK instead of just OFFSET N-1?"**
   - `DENSE_RANK` correctly handles ties. If three people share the top salary, `OFFSET 1` still skips only one row (the second row with the same salary), landing on the second-highest-paid person, not the second distinct salary level. `DENSE_RANK` treats all tied rows as the same rank and jumps to the next distinct value.

2. **"What if N is 0 or negative?"**
   - Add a guard: `IF n < 1 THEN RETURN NULL; END IF;` inside a PL/pgSQL function. In raw SQL, `OFFSET -1` throws an error in PostgreSQL, so always validate the parameter.

**Common mistakes that get you rejected:**
- Using `OFFSET N-1` without `DISTINCT` — produces the Nth row, not the Nth distinct salary.
- Not returning NULL when N exceeds the number of distinct salaries.
- Hardcoding instead of parameterizing when asked to generalize.

---

### Q3: Employees Earning More Than Their Manager
**Company:** LeetCode classic, Razorpay, Swiggy, HCL  **Difficulty:** 🟢  **Frequency:** 🔥🔥🔥  **Round:** OA/Phone

**Question:** Given an `employees` table with a `manager_id` column (self-referencing), write a query to find the names of all employees who earn more than their direct manager.

**What interviewer is testing:** Self-join reasoning, NULL handling (CEO has no manager), and aliasing clarity.

**Ideal Answer:**

```sql
CREATE TABLE employees_mgr (
    id         SERIAL PRIMARY KEY,
    name       VARCHAR(100),
    salary     NUMERIC,
    manager_id INT REFERENCES employees_mgr(id)
);

INSERT INTO employees_mgr (id, name, salary, manager_id) VALUES
    (1, 'CEO Alice',    200000, NULL),
    (2, 'VP Bob',       150000, 1),
    (3, 'Mgr Carol',     90000, 2),
    (4, 'Dev Dave',     100000, 3),   -- earns more than Carol
    (5, 'Dev Eve',       80000, 3),
    (6, 'Mgr Frank',    160000, 2);   -- earns more than Bob

-- ─────────────────────────────────────────────
-- Self-join approach
-- ─────────────────────────────────────────────
SELECT e.name AS employee_name,
       e.salary AS employee_salary,
       m.name AS manager_name,
       m.salary AS manager_salary
FROM   employees_mgr e
JOIN   employees_mgr m ON e.manager_id = m.id
WHERE  e.salary > m.salary;

/*
Expected output:
 employee_name | employee_salary | manager_name | manager_salary
---------------+-----------------+--------------+----------------
 Dev Dave      |          100000 | Mgr Carol    |          90000
 Mgr Frank     |          160000 | VP Bob       |         150000
*/

-- Note: INNER JOIN automatically excludes CEO Alice (manager_id IS NULL)
-- because there is no row in the manager side with id = NULL.
-- If you used LEFT JOIN, CEO's row would appear with NULLs on the manager
-- side and WHERE e.salary > m.salary would be FALSE (NULL comparison).
-- Either way the CEO is correctly excluded.
```

**Follow-up questions the interviewer will ask:**

1. **"What if an employee has no manager (top-level executive)?"**
   - `INNER JOIN` excludes them automatically because `manager_id IS NULL` never matches any `id`. No special handling needed.

2. **"Can you rewrite this without a self-join using a subquery?"**
   ```sql
   SELECT name
   FROM employees_mgr e
   WHERE salary > (
       SELECT salary FROM employees_mgr WHERE id = e.manager_id
   );
   ```
   This correlated subquery runs once per row. Functionally equivalent but generally slower on large tables than the join because the planner cannot use a hash join.

3. **"How would you also show the chain — manager's manager?"**
   - Use a recursive CTE (see Q27).

**Common mistakes that get you rejected:**
- Using `LEFT JOIN` and forgetting that NULLs in WHERE comparisons always evaluate to UNKNOWN (not TRUE), causing incorrect exclusions or inclusions.
- Mixing up the alias direction — joining `e.id = m.manager_id` instead of `e.manager_id = m.id`.

---

### Q4: Find Duplicate Emails, Then Delete Keeping Lowest ID
**Company:** LeetCode classic, Amazon, Infosys, Capgemini  **Difficulty:** 🟢  **Frequency:** 🔥🔥🔥  **Round:** OA/Phone

**Question:** Part A: Find all email addresses that appear more than once in a `person` table. Part B: Delete all duplicate rows, keeping only the row with the smallest `id` for each email.

**What interviewer is testing:** GROUP BY + HAVING for duplicates, DELETE with subquery or CTE, understanding of which rows to keep vs delete.

**Ideal Answer:**

```sql
CREATE TABLE person (
    id    SERIAL PRIMARY KEY,
    email VARCHAR(255)
);

INSERT INTO person (id, email) VALUES
    (1, 'alice@example.com'),
    (2, 'bob@example.com'),
    (3, 'alice@example.com'),
    (4, 'carol@example.com'),
    (5, 'bob@example.com'),
    (6, 'bob@example.com');

-- ─────────────────────────────────────────────
-- Part A: Find duplicate emails
-- ─────────────────────────────────────────────
SELECT email, COUNT(*) AS occurrences
FROM person
GROUP BY email
HAVING COUNT(*) > 1;

/*
 email             | occurrences
-------------------+-------------
 alice@example.com | 2
 bob@example.com   | 3
*/

-- ─────────────────────────────────────────────
-- Part B: Delete duplicates, keep lowest id
-- ─────────────────────────────────────────────

-- Option 1: DELETE with subquery (portable)
DELETE FROM person
WHERE id NOT IN (
    SELECT MIN(id)
    FROM person
    GROUP BY email
);

-- Option 2: DELETE with CTE (cleaner, PostgreSQL idiomatic)
WITH keepers AS (
    SELECT MIN(id) AS keep_id
    FROM person
    GROUP BY email
)
DELETE FROM person
WHERE id NOT IN (SELECT keep_id FROM keepers);

-- Option 3: Self-join DELETE (explicit, shows reasoning)
DELETE FROM person p1
USING person p2
WHERE p1.email = p2.email
  AND p1.id > p2.id;

-- Verify result
SELECT * FROM person ORDER BY id;

/*
 id | email
----+-------------------
  1 | alice@example.com
  2 | bob@example.com
  4 | carol@example.com
*/
```

**Follow-up questions the interviewer will ask:**

1. **"Why might `NOT IN` be dangerous if the subquery can return NULLs?"**
   - If any `MIN(id)` were NULL (impossible here since `id` is NOT NULL, but hypothetically), `NOT IN (NULL, ...)` always evaluates to UNKNOWN for every row, so nothing gets deleted. Always prefer `NOT EXISTS` or an explicit `IS NOT NULL` guard when the subquery might return NULLs.

2. **"What is the difference between your three options in terms of performance?"**
   - The self-join (`DELETE ... USING`) tends to be most efficient because PostgreSQL can use a hash join or merge join on `email`. The `NOT IN` subquery creates a hash of all keeper IDs. The CTE approach is readable and the planner treats it similarly to the subquery approach.

3. **"How would you preview which rows will be deleted before actually deleting them?"**
   - Replace `DELETE` with `SELECT *` using the same `WHERE` clause, or use `RETURNING *` in the `DELETE` statement.

**Common mistakes that get you rejected:**
- Writing `WHERE id NOT IN (SELECT MIN(id) FROM person)` without `GROUP BY email` — this keeps only the single global minimum ID.
- Accidentally deleting the keeper rows (inverting the condition).
- Not testing with the actual data to verify before running destructive DELETE.

---

### Q5: Department Top 3 Salaries — ROW_NUMBER vs DENSE_RANK, Tie Behavior
**Company:** Google, Amazon, Flipkart, Razorpay, Swiggy  **Difficulty:** 🟡  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** Given `employees` and `departments` tables, write a query to find employees who are in the top 3 salary earners within their department. Explain how the result changes if you use `ROW_NUMBER`, `RANK`, or `DENSE_RANK`.

**What interviewer is testing:** Deep understanding of all three ranking functions, tie-breaking semantics, practical ability to choose the right one for a given business requirement, and the ability to explain trade-offs.

**Ideal Answer:**

```sql
-- ─────────────────────────────────────────────
-- Schema and sample data
-- ─────────────────────────────────────────────
CREATE TABLE departments (
    dept_id   SERIAL PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE emp_dept (
    id        SERIAL PRIMARY KEY,
    name      VARCHAR(100),
    salary    NUMERIC,
    dept_id   INT REFERENCES departments(dept_id)
);

INSERT INTO departments (dept_id, dept_name) VALUES
    (1, 'Engineering'),
    (2, 'Sales'),
    (3, 'HR');

INSERT INTO emp_dept (name, salary, dept_id) VALUES
    -- Engineering: will have ties at rank 2
    ('Alice',   120000, 1),
    ('Bob',     110000, 1),
    ('Carol',   110000, 1),   -- tie with Bob
    ('Dave',     95000, 1),
    ('Eve',      85000, 1),
    -- Sales
    ('Frank',    80000, 2),
    ('Grace',    75000, 2),
    ('Heidi',    75000, 2),   -- tie with Grace
    ('Ivan',     70000, 2),
    -- HR
    ('Judy',     60000, 3),
    ('Ken',      55000, 3);

-- ─────────────────────────────────────────────
-- Step 1: Compute all three ranks side by side
-- ─────────────────────────────────────────────
SELECT
    d.dept_name,
    e.name,
    e.salary,
    ROW_NUMBER()  OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS row_num,
    RANK()        OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS rnk,
    DENSE_RANK()  OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS dense_rnk
FROM   emp_dept e
JOIN   departments d USING (dept_id)
ORDER BY d.dept_name, e.salary DESC;

/*
Expected output (Engineering only shown for clarity):
 dept_name   | name  | salary | row_num | rnk | dense_rnk
-------------+-------+--------+---------+-----+-----------
 Engineering | Alice | 120000 |       1 |   1 |         1
 Engineering | Bob   | 110000 |       2 |   2 |         2
 Engineering | Carol | 110000 |       3 |   2 |         2   ← same RANK and DENSE_RANK
 Engineering | Dave  |  95000 |       4 |   4 |         3   ← RANK skips 3; DENSE_RANK does not
 Engineering | Eve   |  85000 |       5 |   5 |         4
*/

-- ─────────────────────────────────────────────
-- Step 2: Filter to "Top 3" — three different results
-- ─────────────────────────────────────────────

-- Using ROW_NUMBER → always exactly 3 rows per dept
-- (ties broken arbitrarily by PostgreSQL — non-deterministic)
SELECT dept_name, name, salary
FROM (
    SELECT
        d.dept_name, e.name, e.salary,
        ROW_NUMBER() OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS rn
    FROM emp_dept e JOIN departments d USING (dept_id)
) ranked
WHERE rn <= 3;

/*
 dept_name   | name  | salary
-------------+-------+--------
 Engineering | Alice | 120000
 Engineering | Bob   | 110000   ← Bob OR Carol, non-deterministic
 Engineering | Carol | 110000
 Sales       | Frank |  80000
 Sales       | Grace |  75000
 Sales       | Heidi |  75000
 HR          | Judy  |  60000
 HR          | Ken   |  55000
*/
-- Note: for Engineering, ROW_NUMBER gives exactly 3 rows,
-- but Bob and Carol are tied — the tiebreaker is undefined.
-- In a real system, add a secondary ORDER BY (e.g., e.id) to make it deterministic.

-- Using RANK → includes all ties, may return MORE than 3 rows per dept
SELECT dept_name, name, salary
FROM (
    SELECT
        d.dept_name, e.name, e.salary,
        RANK() OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS rnk
    FROM emp_dept e JOIN departments d USING (dept_id)
) ranked
WHERE rnk <= 3;

/*
 dept_name   | name  | salary
-------------+-------+--------
 Engineering | Alice | 120000
 Engineering | Bob   | 110000
 Engineering | Carol | 110000
 -- Dave (salary 95000) is EXCLUDED because RANK skips 3, gives Dave rank 4
 Sales       | Frank |  80000
 Sales       | Grace |  75000
 Sales       | Heidi |  75000
 HR          | Judy  |  60000
 HR          | Ken   |  55000
*/
-- RANK skips rank 3 (because two people are at rank 2),
-- so Dave at rank 4 is excluded even though he is the 3rd-distinct salary.

-- Using DENSE_RANK → includes all ties at the top 3 DISTINCT salary levels
SELECT dept_name, name, salary
FROM (
    SELECT
        d.dept_name, e.name, e.salary,
        DENSE_RANK() OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS dr
    FROM emp_dept e JOIN departments d USING (dept_id)
) ranked
WHERE dr <= 3;

/*
 dept_name   | name  | salary
-------------+-------+--------
 Engineering | Alice | 120000
 Engineering | Bob   | 110000
 Engineering | Carol | 110000
 Engineering | Dave  |  95000   ← INCLUDED because dense_rank = 3
 Sales       | Frank |  80000
 Sales       | Grace |  75000
 Sales       | Heidi |  75000
 Sales       | Ivan  |  70000   ← INCLUDED as 3rd distinct salary in Sales
 HR          | Judy  |  60000
 HR          | Ken   |  55000
*/

-- ─────────────────────────────────────────────
-- Step 3: Decision guide — which to use?
-- ─────────────────────────────────────────────
-- ROW_NUMBER: Use when you need exactly N rows per group, no exceptions.
--             Example: "Show 3 featured products per category for a UI card."
--             Risk: ties are broken arbitrarily — add a deterministic tiebreaker.
--
-- RANK:       Use when rank positions must match Olympic-style numbering
--             (1st, 1st, 3rd). "What rank is this employee in their dept?"
--             Gaps appear after ties. Rarely the right choice for top-N filtering.
--
-- DENSE_RANK: Use when you care about distinct salary levels / value tiers.
--             Example: "Top 3 salary bands per department." Most common in
--             interview problems and business reporting.
--
-- ─────────────────────────────────────────────
-- Step 4: Recommended production query (DENSE_RANK + deterministic tiebreak)
-- ─────────────────────────────────────────────
SELECT dept_name, name, salary
FROM (
    SELECT
        d.dept_name,
        e.name,
        e.salary,
        DENSE_RANK() OVER (
            PARTITION BY e.dept_id
            ORDER BY e.salary DESC
        ) AS dr
    FROM emp_dept e
    JOIN departments d USING (dept_id)
) ranked
WHERE dr <= 3
ORDER BY dept_name, salary DESC, name;

-- ─────────────────────────────────────────────
-- Step 5: Alternative without window functions (pre-SQL:2003 style)
-- ─────────────────────────────────────────────
-- Useful if the interviewer asks "how would you do this in MySQL 5.x?"
SELECT d.dept_name, e.name, e.salary
FROM emp_dept e
JOIN departments d ON e.dept_id = d.dept_id
WHERE (
    SELECT COUNT(DISTINCT e2.salary)
    FROM emp_dept e2
    WHERE e2.dept_id = e.dept_id
      AND e2.salary > e.salary
) < 3
ORDER BY d.dept_name, e.salary DESC;

-- The correlated subquery counts how many distinct salaries are strictly
-- above e.salary within the same department. If < 3, this employee is in
-- the top-3 salary tier. This is the DENSE_RANK equivalent without window functions.
-- It is O(n²) and slow on large tables; prefer the window function approach.
```

**Follow-up questions the interviewer will ask:**

1. **"If two employees have the exact same salary and you use ROW_NUMBER, which one gets the lower rank?"**
   - It is non-deterministic unless you add a secondary sort column. PostgreSQL will pick one arbitrarily based on the physical storage order or the internal sort algorithm used. Always add `ORDER BY salary DESC, id ASC` (or another unique column) to make the result repeatable and testable.

2. **"The business requirement says 'top 3 salary tiers'. Which function do you use and why?"**
   - `DENSE_RANK`. The word "tier" means distinct value levels, not row positions. `DENSE_RANK <= 3` gives all employees whose salary falls within the top 3 distinct salary values, regardless of how many employees share each value.

3. **"Can you rewrite the top-N-per-group query using a lateral join instead of a window function?"**
   ```sql
   SELECT d.dept_name, top3.name, top3.salary
   FROM departments d
   CROSS JOIN LATERAL (
       SELECT e.name, e.salary
       FROM emp_dept e
       WHERE e.dept_id = d.dept_id
       ORDER BY e.salary DESC
       LIMIT 3
   ) top3
   ORDER BY d.dept_name, top3.salary DESC;
   ```
   Note: `LIMIT 3` uses `ROW_NUMBER` semantics (exactly 3 rows), not `DENSE_RANK` semantics. For the DENSE_RANK effect via lateral join you would need a more complex predicate.

**Common mistakes that get you rejected:**
- Using `RANK()` for a top-N filter and not being able to explain why it skips ranks.
- Using `ROW_NUMBER()` when ties should be included — losing data silently.
- Forgetting `PARTITION BY dept_id` — ranking across the entire table instead of per department.
- Not being able to articulate the difference between the three functions when asked directly — this is a flagship interview question.
- Writing the correlated-subquery version and claiming it is equivalent to `DENSE_RANK` without verifying the tie semantics.

---

### Q6: Three or More Consecutive Numbers
**Company:** LeetCode 180, Flipkart, Razorpay  **Difficulty:** 🟡  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Given a `logs` table with a sequential `id` and a `num` column, find all numbers that appear at least three times consecutively (in rows with consecutive IDs).

**What interviewer is testing:** Self-join on offset IDs, window function LAG/LEAD approach, understanding of "consecutive" meaning adjacent row IDs (not just the same value appearing multiple times).

**Ideal Answer:**

```sql
CREATE TABLE logs (
    id  SERIAL PRIMARY KEY,
    num INT
);

INSERT INTO logs (id, num) VALUES
    (1, 1),
    (2, 1),
    (3, 1),
    (4, 2),
    (5, 1),
    (6, 2),
    (7, 2);

-- ─────────────────────────────────────────────
-- Approach 1: Self-join on id+1 and id+2
-- ─────────────────────────────────────────────
SELECT DISTINCT l1.num AS consecutive_num
FROM   logs l1
JOIN   logs l2 ON l2.id = l1.id + 1 AND l2.num = l1.num
JOIN   logs l3 ON l3.id = l1.id + 2 AND l3.num = l1.num;

/*
Expected output:
 consecutive_num
-----------------
 1
*/

-- ─────────────────────────────────────────────
-- Approach 2: LAG/LEAD window functions
-- ─────────────────────────────────────────────
SELECT DISTINCT num AS consecutive_num
FROM (
    SELECT num,
           LAG(num, 1)  OVER (ORDER BY id) AS prev1,
           LAG(num, 2)  OVER (ORDER BY id) AS prev2
    FROM logs
) t
WHERE num = prev1 AND num = prev2;

/*
Expected output:
 consecutive_num
-----------------
 1
*/
```

**Follow-up questions the interviewer will ask:**

1. **"What if IDs are not gapless? (Some rows have been deleted.)"**
   - The self-join approach breaks — `l1.id + 1` may not exist. Use `ROW_NUMBER()` to generate a dense sequence first, then apply the self-join or LAG approach on that sequence.

2. **"Generalize to N or more consecutive occurrences."**
   - Use the gap-and-island technique (Q20): assign a group key `ROW_NUMBER() - ROW_NUMBER() OVER (PARTITION BY num ORDER BY id)`, then `HAVING COUNT(*) >= N`.

**Common mistakes that get you rejected:**
- Confusing "consecutive rows by ID" with "the same number appearing N times anywhere in the table" — those are different problems.
- Forgetting `DISTINCT` — the same number might satisfy the condition at multiple positions.

---

### Q7: Consecutive Login Days — Longest Streak Per User
**Company:** Swiggy, PhonePe, Groww, Razorpay  **Difficulty:** 🔴  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Given an `activity` table with `user_id` and `login_date`, find the longest consecutive login streak for each user. Ignore duplicate logins on the same day.

**What interviewer is testing:** Gap-and-island pattern, DATE arithmetic, ability to handle deduplication before the island calculation.

**Ideal Answer:**

```sql
CREATE TABLE activity (
    user_id    INT,
    login_date DATE
);

INSERT INTO activity (user_id, login_date) VALUES
    (1, '2024-01-01'), (1, '2024-01-02'), (1, '2024-01-03'),
    (1, '2024-01-05'),   -- gap here
    (1, '2024-01-06'), (1, '2024-01-07'), (1, '2024-01-07'), -- duplicate
    (2, '2024-01-10'), (2, '2024-01-11'),
    (2, '2024-01-15');

-- Step 1: Deduplicate logins (same user, same day)
-- Step 2: Assign island group = login_date - ROW_NUMBER (as interval)
-- Step 3: COUNT rows per island, take MAX per user

WITH deduped AS (
    SELECT DISTINCT user_id, login_date
    FROM activity
),
numbered AS (
    SELECT
        user_id,
        login_date,
        ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_date) AS rn
    FROM deduped
),
islands AS (
    SELECT
        user_id,
        login_date - (rn * INTERVAL '1 day') AS island_key,
        -- Consecutive dates all produce the same island_key because
        -- each day and its row number advance by 1 simultaneously.
        rn
    FROM numbered
)
SELECT
    user_id,
    MAX(streak_length) AS longest_streak
FROM (
    SELECT user_id, island_key, COUNT(*) AS streak_length
    FROM islands
    GROUP BY user_id, island_key
) streaks
GROUP BY user_id
ORDER BY user_id;

/*
Expected output:
 user_id | longest_streak
---------+----------------
       1 |              3    ← Jan 1, 2, 3
       2 |              2    ← Jan 10, 11
*/
```

**Follow-up questions the interviewer will ask:**

1. **"Why does subtracting row number from date create island groups?"**
   - For consecutive dates, both the date and the row number increase by 1 each step, so `date - rn_days` is constant. A gap in dates (skipping a day) increments the date by 2 but rn by 1, changing the difference and starting a new island group.

2. **"How would you find the date range of each streak (start date and end date)?"**
   ```sql
   SELECT user_id, island_key,
          MIN(login_date) AS streak_start,
          MAX(login_date) AS streak_end,
          COUNT(*) AS streak_length
   FROM islands
   GROUP BY user_id, island_key;
   ```

**Common mistakes that get you rejected:**
- Forgetting to deduplicate before running the island calculation — duplicate dates corrupt the row number sequence and produce wrong island groups.
- Using `login_date - rn` without casting — PostgreSQL requires explicit interval arithmetic: `login_date - (rn * INTERVAL '1 day')` or `login_date - rn::INT` (which works for DATE types).

---

### Q8: Rising Temperature — Days Warmer Than Yesterday
**Company:** LeetCode 197, Amazon OA, Infosys  **Difficulty:** 🟢  **Frequency:** 🔥🔥  **Round:** OA/Phone

**Question:** Given a `weather` table with `id`, `record_date`, and `temperature`, find the IDs of all days where the temperature was higher than the previous day's temperature.

**What interviewer is testing:** Date-based self-join, LAG window function, handling non-consecutive dates.

**Ideal Answer:**

```sql
CREATE TABLE weather (
    id          SERIAL PRIMARY KEY,
    record_date DATE,
    temperature NUMERIC
);

INSERT INTO weather (id, record_date, temperature) VALUES
    (1, '2024-01-01', 10),
    (2, '2024-01-02', 25),   -- warmer than day 1
    (3, '2024-01-03', 20),
    (4, '2024-01-04', 30),   -- warmer than day 3
    (5, '2024-01-06', 35);   -- warmer than day 4, but Jan 5 is missing

-- ─────────────────────────────────────────────
-- Approach 1: Self-join on date - 1
-- ─────────────────────────────────────────────
SELECT w1.id
FROM   weather w1
JOIN   weather w2 ON w2.record_date = w1.record_date - INTERVAL '1 day'
WHERE  w1.temperature > w2.temperature;

/*
Expected output:
 id
----
  2
  4
-- Note: id=5 (Jan 6) is NOT returned because Jan 5 is missing,
-- so there is no w2 row to join to.
*/

-- ─────────────────────────────────────────────
-- Approach 2: LAG window function
-- ─────────────────────────────────────────────
SELECT id
FROM (
    SELECT id,
           temperature,
           record_date,
           LAG(temperature) OVER (ORDER BY record_date) AS prev_temp,
           LAG(record_date) OVER (ORDER BY record_date) AS prev_date
    FROM weather
) t
WHERE temperature > prev_temp
  AND record_date = prev_date + INTERVAL '1 day';
  -- Guard: only compare if the previous row is actually yesterday,
  -- not a row from two days ago (handles gaps in dates).

/*
Expected output:
 id
----
  2
  4
*/
```

**Follow-up questions the interviewer will ask:**

1. **"What happens with the LAG approach if there are gaps in dates?"**
   - Without the date-continuity guard (`record_date = prev_date + INTERVAL '1 day'`), LAG returns the previous row regardless of the date gap. Jan 6 would incorrectly compare against Jan 4. Always add the date check.

2. **"Which approach is faster?"**
   - LAG scans the table once; self-join may require two scans and a join. On large tables with an index on `record_date`, both are efficient, but LAG is typically faster and clearer.

**Common mistakes that get you rejected:**
- Using `LAG` without the date-continuity check — produces wrong results when dates have gaps.
- Joining on `w1.id = w2.id - 1` instead of on dates — assumes IDs are consecutive and gap-free.

---

### Q9: Tree Node Classification — Root, Inner, Leaf
**Company:** LeetCode 608, Flipkart, Goldman Sachs  **Difficulty:** 🟡  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Given a `tree` table with `id` and `p_id` (parent ID), classify each node as 'Root' (no parent), 'Leaf' (no children), or 'Inner' (has both parent and children).

**What interviewer is testing:** Self-referencing table reasoning, CASE expressions, subquery membership checks.

**Ideal Answer:**

```sql
CREATE TABLE tree (
    id   INT PRIMARY KEY,
    p_id INT   -- NULL means root
);

INSERT INTO tree (id, p_id) VALUES
    (1, NULL),  -- root
    (2, 1),
    (3, 1),
    (4, 2),
    (5, 2);
-- Tree structure:
--       1 (Root)
--      / \
--     2   3 (Leaf)
--    / \
--   4   5 (both Leaves)

SELECT
    id,
    CASE
        WHEN p_id IS NULL THEN 'Root'
        WHEN id IN (SELECT DISTINCT p_id FROM tree WHERE p_id IS NOT NULL) THEN 'Inner'
        ELSE 'Leaf'
    END AS node_type
FROM tree
ORDER BY id;

/*
Expected output:
 id | node_type
----+-----------
  1 | Root
  2 | Inner
  3 | Leaf
  4 | Leaf
  5 | Leaf
*/

-- Alternative using LEFT JOIN (avoids subquery scan per row)
SELECT t.id,
       CASE
           WHEN t.p_id IS NULL THEN 'Root'
           WHEN child.id IS NOT NULL THEN 'Inner'
           ELSE 'Leaf'
       END AS node_type
FROM tree t
LEFT JOIN tree child ON child.p_id = t.id
ORDER BY t.id;
-- The LEFT JOIN checks if any row has t.id as its parent.
-- If yes → t is Inner (or Root, checked first). If no → Leaf.
```

**Follow-up questions the interviewer will ask:**

1. **"Can a root node also be a leaf (single-node tree)?"**
   - Yes. Check for `p_id IS NULL` AND `id NOT IN (SELECT p_id ...)`. In the CASE, put the Root check first so it takes precedence.

2. **"How would you traverse the entire tree (BFS/DFS) in SQL?"**
   - Use a recursive CTE (see Q27/Q28).

**Common mistakes that get you rejected:**
- Not handling `NULL` in `IN (SELECT p_id ...)` — if any `p_id` is NULL and you do `id NOT IN (subquery returning NULLs)`, the result is always UNKNOWN. The `WHERE p_id IS NOT NULL` inside the subquery is critical.

---

### Q10: Rank Scores Without RANK() — Simulate DENSE_RANK
**Company:** LeetCode 178, Accenture, Wipro, Cognizant  **Difficulty:** 🟡  **Frequency:** 🔥🔥  **Round:** OA/Phone

**Question:** Given a `scores` table with `id` and `score`, rank each score from highest to lowest. Scores with the same value get the same rank, and there should be no gaps (DENSE_RANK behavior). Do not use any ranking window function.

**What interviewer is testing:** Correlated subquery reasoning, ability to manually simulate window functions, understanding of what DENSE_RANK actually computes.

**Ideal Answer:**

```sql
CREATE TABLE scores (
    id    SERIAL PRIMARY KEY,
    score NUMERIC(5,2)
);

INSERT INTO scores (id, score) VALUES
    (1, 3.50),
    (2, 3.65),
    (3, 4.00),
    (4, 3.85),
    (5, 4.00),
    (6, 3.65);

-- ─────────────────────────────────────────────
-- Correlated subquery: count distinct scores greater than current
-- ─────────────────────────────────────────────
SELECT
    score,
    (
        SELECT COUNT(DISTINCT s2.score)
        FROM scores s2
        WHERE s2.score > s1.score
    ) + 1 AS rank
FROM scores s1
ORDER BY score DESC;

/*
Expected output:
 score | rank
-------+------
  4.00 |    1
  4.00 |    1
  3.85 |    2
  3.65 |    3
  3.65 |    3
  3.50 |    4
*/

-- Logic:
-- rank = (number of distinct scores strictly greater than this score) + 1
-- score 4.00: 0 scores above → rank 1
-- score 3.85: 1 distinct score above (4.00) → rank 2
-- score 3.65: 2 distinct scores above → rank 3
-- score 3.50: 3 distinct scores above → rank 4

-- ─────────────────────────────────────────────
-- For comparison: the window function version
-- ─────────────────────────────────────────────
SELECT score,
       DENSE_RANK() OVER (ORDER BY score DESC) AS rank
FROM scores
ORDER BY score DESC;
-- Produces identical output. The correlated subquery is the
-- manual implementation of exactly what DENSE_RANK computes.
```

**Follow-up questions the interviewer will ask:**

1. **"This correlated subquery runs once per row — what is the time complexity?"**
   - O(n²) in the worst case. On large tables, this is significantly slower than the window function, which is O(n log n) due to the sort. An index on `score` can help the subquery (each count becomes an index range scan) but the window function is still faster in practice.

2. **"How would you simulate ROW_NUMBER without window functions?"**
   - `(SELECT COUNT(*) FROM scores s2 WHERE s2.score > s1.score OR (s2.score = s1.score AND s2.id < s1.id)) + 1` — count rows that come strictly before in a composite ordering.

**Common mistakes that get you rejected:**
- Using `COUNT(*)` instead of `COUNT(DISTINCT score)` — produces RANK behavior (with gaps) instead of DENSE_RANK behavior.
- Off-by-one error: forgetting the `+ 1` makes the highest score rank 0.

---

### Q11: Trips and Users — Cancellation Rate
**Company:** LeetCode 262, Uber, Ola, Rapido  **Difficulty:** 🔴  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Given a `trips` table and a `users` table, compute the cancellation rate for each day between `2013-10-01` and `2013-10-03`. Exclude trips where the client or driver is banned. Round to 2 decimal places.

**What interviewer is testing:** Multi-table joins, conditional aggregation with CASE, ROUND function, date filtering, and filtering via a join rather than a subquery.

**Ideal Answer:**

```sql
CREATE TABLE users_trips (
    users_id  INT PRIMARY KEY,
    banned    VARCHAR(3),   -- 'Yes' or 'No'
    role      VARCHAR(10)   -- 'client' or 'driver'
);

CREATE TABLE trips (
    id          SERIAL PRIMARY KEY,
    client_id   INT REFERENCES users_trips(users_id),
    driver_id   INT REFERENCES users_trips(users_id),
    city_id     INT,
    status      VARCHAR(30),  -- 'completed', 'cancelled_by_driver', 'cancelled_by_client'
    request_at  DATE
);

INSERT INTO users_trips VALUES
    (1, 'No',  'client'), (2, 'Yes', 'client'),
    (3, 'No',  'client'), (4, 'No',  'driver'),
    (10, 'No', 'driver'), (11, 'No', 'driver'), (12, 'No', 'driver');

INSERT INTO trips (id, client_id, driver_id, city_id, status, request_at) VALUES
    (1,  1, 10, 1, 'completed',           '2013-10-01'),
    (2,  2, 11, 1, 'cancelled_by_driver', '2013-10-01'),  -- client banned
    (3,  3, 12, 6, 'completed',           '2013-10-01'),
    (4,  4, 13, 6, 'cancelled_by_client', '2013-10-01'),  -- driver doesn't exist, skip
    (5,  1, 10, 1, 'completed',           '2013-10-02'),
    (6,  2, 11, 6, 'completed',           '2013-10-02'),  -- client banned
    (7,  3, 12, 6, 'completed',           '2013-10-03'),
    (8,  2, 12, 12,'completed',           '2013-10-03');  -- client banned

-- ─────────────────────────────────────────────
-- Solution: filter banned users via JOIN, then aggregate
-- ─────────────────────────────────────────────
SELECT
    t.request_at AS "Day",
    ROUND(
        SUM(CASE WHEN t.status LIKE 'cancelled%' THEN 1.0 ELSE 0 END)
        / COUNT(*),
        2
    ) AS "Cancellation Rate"
FROM trips t
JOIN users_trips c ON c.users_id = t.client_id AND c.banned = 'No'
JOIN users_trips d ON d.users_id = t.driver_id AND d.banned = 'No'
WHERE t.request_at BETWEEN '2013-10-01' AND '2013-10-03'
GROUP BY t.request_at
ORDER BY t.request_at;

/*
Expected output:
 Day        | Cancellation Rate
------------+-------------------
 2013-10-01 |              0.33
 2013-10-02 |              0.00
 2013-10-03 |              0.00
*/
```

**Follow-up questions the interviewer will ask:**

1. **"Why use JOIN with `banned = 'No'` instead of `WHERE client_id NOT IN (SELECT users_id FROM users WHERE banned = 'Yes')`?"**
   - The JOIN approach is cleaner and avoids the NULL-in-NOT-IN pitfall. It also lets the planner use hash join more efficiently. Both approaches are correct here, but JOIN is preferred.

2. **"Why `1.0` in the CASE expression?"**
   - `SUM(CASE WHEN ... THEN 1 ELSE 0 END)` would produce an INTEGER sum divided by an INTEGER count → integer division → truncation. Using `1.0` promotes the numerator to NUMERIC so the division is fractional. Alternatively, cast: `SUM(...)::NUMERIC / COUNT(*)`.

**Common mistakes that get you rejected:**
- Forgetting to filter banned users — the most common error.
- Integer division producing 0 for all rows.
- Not using `ROUND(..., 2)` — the problem explicitly requires 2 decimal places.

---

### Q12: Exchange Seats — Swap Adjacent Rows
**Company:** LeetCode 626, Flipkart, Walmart Labs  **Difficulty:** 🟡  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Given a `seat` table with `id` (1-based, sequential) and `student` name, swap adjacent students so that student in seat 1 swaps with seat 2, seat 3 swaps with seat 4, etc. If there is an odd number of students, the last student stays in place.

**What interviewer is testing:** CASE + MOD arithmetic for conditional logic, handling the odd-seat edge case.

**Ideal Answer:**

```sql
CREATE TABLE seat (
    id      SERIAL PRIMARY KEY,
    student VARCHAR(100)
);

INSERT INTO seat (id, student) VALUES
    (1, 'Alice'),
    (2, 'Bob'),
    (3, 'Carol'),
    (4, 'Dave'),
    (5, 'Eve');    -- odd number: Eve stays in seat 5

SELECT
    CASE
        WHEN MOD(id, 2) = 1 AND id + 1 <= (SELECT MAX(id) FROM seat)
            THEN id + 1             -- odd seat → bump up by 1
        WHEN MOD(id, 2) = 0
            THEN id - 1             -- even seat → pull back by 1
        ELSE id                     -- last odd seat → stay
    END AS id,
    student
FROM seat
ORDER BY 1;

/*
Expected output:
 id | student
----+---------
  1 | Bob
  2 | Alice
  3 | Dave
  4 | Carol
  5 | Eve
*/
```

**Follow-up questions the interviewer will ask:**

1. **"How would you do this using window functions (LEAD/LAG)?"**
   ```sql
   SELECT id,
          COALESCE(
              CASE WHEN MOD(id, 2) = 1 THEN LEAD(student) OVER (ORDER BY id) END,
              CASE WHEN MOD(id, 2) = 0 THEN LAG(student)  OVER (ORDER BY id) END
          ) AS student
   FROM seat
   ORDER BY id;
   ```

2. **"What if IDs are not sequential (gaps exist)?"**
   - Use `ROW_NUMBER() OVER (ORDER BY id)` to create a gapless sequence, apply the swap logic on that, then join back on the original `id`.

**Common mistakes that get you rejected:**
- Forgetting the edge case for the last seat when there are an odd number of students.
- Outputting the rows in the wrong order (not ordering by the new ID).

---

### Q13: Customers Who Never Placed an Order
**Company:** LeetCode 183, Amazon, Myntra, Nykaa  **Difficulty:** 🟢  **Frequency:** 🔥🔥🔥  **Round:** OA/Phone

**Question:** Given `customers` and `orders` tables, find all customers who have never placed an order. Implement all three approaches: LEFT JOIN + IS NULL, NOT EXISTS, and NOT IN. Explain the performance differences.

**What interviewer is testing:** Knowledge of all three anti-join patterns, understanding of NULL semantics in NOT IN, and practical performance awareness.

**Ideal Answer:**

```sql
CREATE TABLE customers (
    id   SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE orders (
    id          SERIAL PRIMARY KEY,
    customer_id INT REFERENCES customers(id),
    amount      NUMERIC
);

INSERT INTO customers VALUES
    (1, 'Alice'), (2, 'Bob'), (3, 'Carol'), (4, 'Dave');

INSERT INTO orders (customer_id, amount) VALUES
    (1, 250.00),
    (1, 180.00),
    (3, 500.00);

-- ─────────────────────────────────────────────
-- Approach 1: LEFT JOIN + IS NULL (exclusion join / anti-join)
-- ─────────────────────────────────────────────
SELECT c.id, c.name
FROM   customers c
LEFT JOIN orders o ON o.customer_id = c.id
WHERE  o.id IS NULL;

/*
Expected output:
 id | name
----+------
  2 | Bob
  4 | Dave
*/

-- ─────────────────────────────────────────────
-- Approach 2: NOT EXISTS (correlated subquery)
-- ─────────────────────────────────────────────
SELECT c.id, c.name
FROM   customers c
WHERE  NOT EXISTS (
    SELECT 1 FROM orders o WHERE o.customer_id = c.id
);

-- ─────────────────────────────────────────────
-- Approach 3: NOT IN (subquery)
-- ─────────────────────────────────────────────
SELECT c.id, c.name
FROM   customers c
WHERE  c.id NOT IN (
    SELECT customer_id FROM orders WHERE customer_id IS NOT NULL
);

-- CRITICAL: The WHERE customer_id IS NOT NULL guard is MANDATORY.
-- If orders.customer_id contains even ONE NULL value (orphaned order):
--   NOT IN (1, 3, NULL) → for every c.id, evaluates as:
--     c.id != 1 AND c.id != 3 AND c.id != NULL
--   The last term is UNKNOWN → entire expression is UNKNOWN → row excluded.
-- Result: zero rows returned — every customer "might have" an order.
-- This is the most dangerous SQL trap in practice.

-- ─────────────────────────────────────────────
-- Performance notes
-- ─────────────────────────────────────────────
-- LEFT JOIN + IS NULL:
--   PostgreSQL planner typically converts this to a hash anti-join.
--   O(n + m) where n = customers, m = orders. Usually fastest.
--   Requires the join column to be indexed (customer_id) for large tables.
--
-- NOT EXISTS:
--   PostgreSQL also converts to hash anti-join in most cases.
--   Semantically clearest — reads as "no order exists for this customer."
--   Short-circuits: stops scanning orders once a match is found.
--   Preferred for readability and safety (NULL-safe by construction).
--
-- NOT IN:
--   Historically could not be converted to an anti-join in all databases.
--   In PostgreSQL 9.4+, the planner often converts it to a hash anti-join
--   when the NOT IN list is a subquery. But the NULL trap makes it risky.
--   Avoid NOT IN with subqueries that may produce NULLs — use NOT EXISTS.
```

**Follow-up questions the interviewer will ask:**

1. **"Which approach would you recommend in production and why?"**
   - `NOT EXISTS` for safety and clarity. It is NULL-safe by construction (the subquery either finds a row or it doesn't; NULLs in the subquery columns don't poison the result). PostgreSQL's planner converts it to a hash anti-join with the same performance as LEFT JOIN + IS NULL.

2. **"When does NOT IN return zero rows even when you expect results?"**
   - When the subquery returns ANY NULL value. `x NOT IN (1, 2, NULL)` evaluates as `x <> 1 AND x <> 2 AND x <> NULL`. Since `x <> NULL` is always UNKNOWN, the whole expression is UNKNOWN (never TRUE), filtering out every row.

3. **"Is there a performance difference between LEFT JOIN and NOT EXISTS in PostgreSQL?"**
   - PostgreSQL's query planner is smart enough to convert both to the same physical plan (hash anti-join) in most cases. Run `EXPLAIN ANALYZE` on both — you'll often see identical plans. The difference matters more in MySQL or older SQL Server versions.

**Common mistakes that get you rejected:**
- Using `NOT IN` without the `IS NOT NULL` guard — passes on clean test data but fails in production where NULLs exist.
- Using `LEFT JOIN` but checking `c.id IS NULL` instead of `o.id IS NULL` (or another orders column) — `c.id` can never be NULL since it's a PRIMARY KEY.
- Not knowing all three approaches when explicitly asked.

---

### Q14: Department with Highest Salary
**Company:** LeetCode 184, Amazon, Flipkart, Accenture  **Difficulty:** 🟢  **Frequency:** 🔥🔥🔥  **Round:** OA/Phone

**Question:** Given `employee` and `department` tables, find the employee(s) with the highest salary in each department.

**What interviewer is testing:** GROUP BY aggregation, joining aggregated results back to detail rows, handling ties at the maximum.

**Ideal Answer:**

```sql
-- Reusing emp_dept and departments tables from Q5.

-- ─────────────────────────────────────────────
-- Approach 1: GROUP BY + JOIN
-- ─────────────────────────────────────────────
SELECT d.dept_name AS Department,
       e.name      AS Employee,
       e.salary    AS Salary
FROM   emp_dept e
JOIN   departments d ON d.dept_id = e.dept_id
JOIN (
    SELECT dept_id, MAX(salary) AS max_salary
    FROM emp_dept
    GROUP BY dept_id
) dept_max ON dept_max.dept_id = e.dept_id
          AND dept_max.max_salary = e.salary
ORDER BY d.dept_name, e.name;

/*
Expected output (using Q5 data):
 Department  | Employee | Salary
-------------+----------+--------
 Engineering | Alice    | 120000
 HR          | Judy     |  60000
 Sales       | Frank    |  80000
*/

-- ─────────────────────────────────────────────
-- Approach 2: DENSE_RANK (handles ties elegantly)
-- ─────────────────────────────────────────────
SELECT dept_name AS Department, name AS Employee, salary AS Salary
FROM (
    SELECT d.dept_name, e.name, e.salary,
           DENSE_RANK() OVER (PARTITION BY e.dept_id ORDER BY e.salary DESC) AS dr
    FROM emp_dept e JOIN departments d USING (dept_id)
) ranked
WHERE dr = 1
ORDER BY dept_name, name;
```

**Follow-up questions the interviewer will ask:**

1. **"What if two employees in the same department have the same maximum salary?"**
   - The GROUP BY + JOIN approach returns both (because both match `max_salary`). The DENSE_RANK approach also returns both (both get rank 1). Both are correct — all co-top earners should be included.

2. **"Can you use a window function without a subquery to solve this?"**
   - No. Window functions cannot be used directly in WHERE clauses. You always need to wrap the window function in a subquery or CTE and filter in the outer query.

**Common mistakes that get you rejected:**
- Using `MAX(salary)` in the SELECT without GROUP BY and without a subquery — returns a single global max.
- Joining the derived table with `=` on salary but forgetting to also join on `dept_id` — leaks salaries across departments.

---

### Q15: Employees Present in Every Department
**Company:** Groww, CRED, Razorpay  **Difficulty:** 🔴  **Frequency:** 🔥  **Round:** Onsite

**Question:** Given a table `emp_roles` with `employee_id` and `dept_id`, find all employees who have worked in (or are assigned to) every department.

**What interviewer is testing:** Relational division problem, HAVING COUNT(DISTINCT ...) = total approach, and the harder NOT EXISTS / double-negation approach.

**Ideal Answer:**

```sql
CREATE TABLE emp_roles (
    employee_id INT,
    dept_id     INT,
    PRIMARY KEY (employee_id, dept_id)
);

INSERT INTO emp_roles VALUES
    (1, 1), (1, 2), (1, 3),   -- employee 1 in all 3 depts
    (2, 1), (2, 2),            -- employee 2 missing dept 3
    (3, 1), (3, 2), (3, 3);   -- employee 3 in all 3 depts

-- ─────────────────────────────────────────────
-- Approach 1: HAVING COUNT(DISTINCT dept_id) = total depts
-- ─────────────────────────────────────────────
SELECT employee_id
FROM emp_roles
GROUP BY employee_id
HAVING COUNT(DISTINCT dept_id) = (SELECT COUNT(*) FROM departments);

/*
Expected output:
 employee_id
-------------
           1
           3
*/

-- ─────────────────────────────────────────────
-- Approach 2: Relational division — NOT EXISTS / double negation
-- "Find employees for whom there does NOT EXIST a dept they are NOT in"
-- ─────────────────────────────────────────────
SELECT DISTINCT employee_id
FROM emp_roles e1
WHERE NOT EXISTS (
    SELECT dept_id FROM departments d
    WHERE NOT EXISTS (
        SELECT 1 FROM emp_roles e2
        WHERE e2.employee_id = e1.employee_id
          AND e2.dept_id = d.dept_id
    )
);
```

**Follow-up questions the interviewer will ask:**

1. **"Which approach is more generalizable?"**
   - The `HAVING COUNT(DISTINCT ...)` approach is simpler and sufficient when the set to divide by is a single column. The double-NOT EXISTS (relational division) approach generalizes to cases where the "requirement" itself is a multi-column set or a complex condition.

2. **"What if the departments table gains a new department? Does your query still work?"**
   - Yes — `(SELECT COUNT(*) FROM departments)` is dynamic. The query recalculates the total at runtime.

**Common mistakes that get you rejected:**
- Using `COUNT(dept_id)` without `DISTINCT` — an employee with two rows in the same department could falsely satisfy the count.
- Hardcoding the total number of departments.

---

### Q16: Running Total (Cumulative Sum)
**Company:** Razorpay, PhonePe, Groww, Stripe, PayPal  **Difficulty:** 🟡  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** Given a `transactions` table with `transaction_id`, `user_id`, `amount`, and `transaction_date`, compute the cumulative sum of `amount` per user ordered by date. Also explain the full window frame syntax and its variants.

**What interviewer is testing:** Deep understanding of window frame clauses, partitioning, ordering inside windows, and the distinction between different frame boundaries. This is a flagship window function question.

**Ideal Answer:**

```sql
-- ─────────────────────────────────────────────
-- Schema and data
-- ─────────────────────────────────────────────
CREATE TABLE transactions (
    transaction_id   SERIAL PRIMARY KEY,
    user_id          INT,
    amount           NUMERIC,
    transaction_date DATE
);

INSERT INTO transactions (user_id, amount, transaction_date) VALUES
    (1, 100, '2024-01-01'),
    (1, 200, '2024-01-03'),
    (1,  50, '2024-01-03'),   -- same date as previous: both included in running total
    (1, 300, '2024-01-05'),
    (2, 500, '2024-01-01'),
    (2, 150, '2024-01-04');

-- ─────────────────────────────────────────────
-- Core solution: SUM with ROWS UNBOUNDED PRECEDING
-- ─────────────────────────────────────────────
SELECT
    transaction_id,
    user_id,
    transaction_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY transaction_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total
FROM transactions
ORDER BY user_id, transaction_date, transaction_id;

/*
Expected output:
 transaction_id | user_id | transaction_date | amount | running_total
----------------+---------+------------------+--------+---------------
              1 |       1 | 2024-01-01       |    100 |           100
              2 |       1 | 2024-01-03       |    200 |           300
              3 |       1 | 2024-01-03       |     50 |           350
              4 |       1 | 2024-01-05       |    300 |           650
              5 |       2 | 2024-01-01       |    500 |           500
              6 |       2 | 2024-01-04       |    150 |           650
*/

-- ─────────────────────────────────────────────
-- Window frame anatomy: full breakdown
-- ─────────────────────────────────────────────
--
-- SUM(amount) OVER (
--     PARTITION BY user_id              ← restart the calculation for each user
--     ORDER BY transaction_date         ← defines the row ordering within the partition
--     ROWS BETWEEN                      ← frame mode: ROWS or RANGE
--         UNBOUNDED PRECEDING           ← start from the first row of the partition
--         AND                           ← through
--         CURRENT ROW                   ← up to and including this row
-- )
--
-- ─────────────────────────────────────────────
-- ROWS vs RANGE — critical difference
-- ─────────────────────────────────────────────
--
-- ROWS mode: counts physical rows
-- RANGE mode: includes all rows with the same ORDER BY value as CURRENT ROW
--
-- In our data, Jan 3 has two rows (amount 200 and 50).
-- With ROWS: each row gets an independent running total.
--   Jan 3 row1 (200) running total = 100 + 200 = 300
--   Jan 3 row2 (50)  running total = 100 + 200 + 50 = 350
-- With RANGE: both Jan 3 rows are treated as "current" simultaneously.
--   Both get the same running total = 100 + 200 + 50 = 350

-- Demonstrate RANGE behavior:
SELECT
    transaction_id,
    user_id,
    transaction_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY transaction_date
        RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        -- RANGE: all rows with same date as current row are included together
    ) AS running_total_range
FROM transactions
ORDER BY user_id, transaction_date, transaction_id;

/*
 transaction_id | user_id | transaction_date | amount | running_total_range
----------------+---------+------------------+--------+---------------------
              1 |       1 | 2024-01-01       |    100 |                 100
              2 |       1 | 2024-01-03       |    200 |                 350   ← both Jan 3 rows
              3 |       1 | 2024-01-03       |     50 |                 350   ← included together
              4 |       1 | 2024-01-05       |    300 |                 650
*/

-- Key insight: when ORDER BY has ties and you need each row to be independent,
-- use ROWS. When you want tied rows to receive the same cumulative total, use RANGE.

-- ─────────────────────────────────────────────
-- PostgreSQL default when frame is omitted
-- ─────────────────────────────────────────────
-- If you write: SUM(amount) OVER (PARTITION BY user_id ORDER BY date)
-- PostgreSQL implicitly uses:
--   RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
-- This is RANGE mode — which means tied dates get the same cumulative total.
-- This often surprises developers. ALWAYS specify ROWS or RANGE explicitly.

-- ─────────────────────────────────────────────
-- All frame boundary options
-- ─────────────────────────────────────────────
--
-- Frame start options:
--   UNBOUNDED PRECEDING  — from the first row of the partition
--   N PRECEDING          — N rows before current (ROWS mode) or N units before (RANGE mode)
--   CURRENT ROW          — from the current row
--
-- Frame end options:
--   CURRENT ROW          — through the current row (default end)
--   N FOLLOWING          — N rows after current
--   UNBOUNDED FOLLOWING  — through the last row of the partition
--
-- Example: sum of previous 2 rows and current row (sliding window of 3)
SELECT
    transaction_id,
    user_id,
    transaction_date,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY transaction_date, transaction_id  -- tie-break by id for determinism
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS rolling_3_sum
FROM transactions
ORDER BY user_id, transaction_date, transaction_id;

-- Example: sum of entire partition (grand total per user, repeated on each row)
SELECT
    transaction_id,
    user_id,
    amount,
    SUM(amount) OVER (PARTITION BY user_id) AS total_per_user
FROM transactions;
-- No ORDER BY in the OVER clause → no frame → computes over the entire partition.

-- ─────────────────────────────────────────────
-- Practical variant: cumulative percentage
-- ─────────────────────────────────────────────
SELECT
    transaction_id,
    user_id,
    amount,
    SUM(amount) OVER (
        PARTITION BY user_id
        ORDER BY transaction_date, transaction_id
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total,
    ROUND(
        100.0 * SUM(amount) OVER (
            PARTITION BY user_id
            ORDER BY transaction_date, transaction_id
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) / SUM(amount) OVER (PARTITION BY user_id),
        2
    ) AS cumulative_pct
FROM transactions
ORDER BY user_id, transaction_date, transaction_id;

/*
Sample (user_id = 1, total = 650):
 transaction_id | user_id | amount | running_total | cumulative_pct
----------------+---------+--------+---------------+----------------
              1 |       1 |    100 |           100 |          15.38
              2 |       1 |    200 |           300 |          46.15
              3 |       1 |     50 |           350 |          53.85
              4 |       1 |    300 |           650 |         100.00
*/

-- ─────────────────────────────────────────────
-- Named window (avoid repeating the definition)
-- ─────────────────────────────────────────────
SELECT
    transaction_id,
    user_id,
    amount,
    SUM(amount)   OVER w AS running_total,
    AVG(amount)   OVER w AS running_avg,
    COUNT(*)      OVER w AS running_count
FROM transactions
WINDOW w AS (
    PARTITION BY user_id
    ORDER BY transaction_date, transaction_id
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
)
ORDER BY user_id, transaction_date, transaction_id;
-- Named windows reduce duplication and make complex queries more readable.
```

**Follow-up questions the interviewer will ask:**

1. **"What is the difference between `ROWS UNBOUNDED PRECEDING` and `RANGE UNBOUNDED PRECEDING` when there are ties in the ORDER BY column?"**
   - `ROWS`: the frame ends at the physical current row. If two rows share the same date, the first gets a running total that excludes the second; the second includes both. Each row is independent.
   - `RANGE`: the frame includes all rows with the same ORDER BY value. If two rows share the same date, both get a running total that includes both of them — they are treated as a peer group. This is PostgreSQL's default when you specify `ORDER BY` but omit the frame clause.
   - Practical implication: for financial running totals where each transaction is independent (even on the same day), always use `ROWS`. For reports where same-date entries are a single logical event, `RANGE` may be appropriate.

2. **"How would you compute a 7-day moving average using window frames?"**
   - See Q18. Short answer: `AVG(amount) OVER (PARTITION BY user_id ORDER BY transaction_date ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)`.

3. **"Can you use SUM OVER without ORDER BY?"**
   - Yes: `SUM(amount) OVER (PARTITION BY user_id)` computes the total for the entire partition and repeats it on every row in that partition. There is no running accumulation — just a repeated total. This is useful for "what fraction of user's total is this transaction?" calculations.

4. **"What is a named window and why would you use it?"**
   - A `WINDOW` clause lets you define a window specification once and reference it by name across multiple window function calls in the same SELECT. It avoids repetition and makes queries easier to maintain. Shown in the last example above.

5. **"Is there any performance concern with multiple window functions in the same query?"**
   - Each distinct window specification (different PARTITION BY, ORDER BY, or frame) requires a separate sort pass over the data. If multiple window functions share the same specification, PostgreSQL processes them in a single pass. This is another reason to use named windows — it signals to the planner that the windows are identical and can share the sort.

**Common mistakes that get you rejected:**
- Omitting the frame clause and assuming ROWS behavior — the implicit default is RANGE, which causes unexpected ties behavior.
- Forgetting `PARTITION BY` when computing per-user running totals — computes a global running total across all users.
- Not adding a tiebreaker (e.g., `transaction_id`) to the `ORDER BY` inside the OVER clause when dates can repeat — results become non-deterministic.
- Confusing the window ORDER BY (which determines frame membership) with the query-level ORDER BY (which sorts the final output) — they are independent.
- Using `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` for a running total — this always gives the grand total, not a cumulative sum.

---

### Q17: Year-over-Year Growth Percentage
**Company:** Razorpay, Groww, PhonePe, Stripe  **Difficulty:** 🟡  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Given a `revenue` table with `product`, `year`, and `amount`, compute the year-over-year growth percentage for each product per year. Show NULL (or 0%) for the first year of each product.

**What interviewer is testing:** LAG with PARTITION BY, NULL handling on the first period, percentage calculation, and safe division (avoiding divide-by-zero).

**Ideal Answer:**

```sql
CREATE TABLE revenue (
    product VARCHAR(100),
    year    INT,
    amount  NUMERIC,
    PRIMARY KEY (product, year)
);

INSERT INTO revenue VALUES
    ('Widget', 2021, 100000),
    ('Widget', 2022, 120000),
    ('Widget', 2023, 108000),
    ('Widget', 2024, 135000),
    ('Gadget', 2022,  50000),
    ('Gadget', 2023,  75000),
    ('Gadget', 2024,  70000);

SELECT
    product,
    year,
    amount,
    LAG(amount) OVER (PARTITION BY product ORDER BY year) AS prev_year_amount,
    ROUND(
        CASE
            WHEN LAG(amount) OVER (PARTITION BY product ORDER BY year) IS NULL THEN NULL
            WHEN LAG(amount) OVER (PARTITION BY product ORDER BY year) = 0     THEN NULL
            ELSE 100.0
                 * (amount - LAG(amount) OVER (PARTITION BY product ORDER BY year))
                 / LAG(amount) OVER (PARTITION BY product ORDER BY year)
        END,
        2
    ) AS yoy_growth_pct
FROM revenue
ORDER BY product, year;

/*
 product | year | amount  | prev_year_amount | yoy_growth_pct
---------+------+---------+-----------------+----------------
 Gadget  | 2022 |   50000 |          (null) |         (null)
 Gadget  | 2023 |   75000 |           50000 |          50.00
 Gadget  | 2024 |   70000 |           75000 |          -6.67
 Widget  | 2021 |  100000 |          (null) |         (null)
 Widget  | 2022 |  120000 |          100000 |          20.00
 Widget  | 2023 |  108000 |          120000 |         -10.00
 Widget  | 2024 |  135000 |          108000 |          25.00
*/

-- Cleaner version using a CTE to avoid repeating the LAG expression:
WITH lagged AS (
    SELECT
        product, year, amount,
        LAG(amount) OVER (PARTITION BY product ORDER BY year) AS prev_amount
    FROM revenue
)
SELECT
    product,
    year,
    amount,
    prev_amount,
    ROUND(
        NULLIF(prev_amount, 0)
        |> (SELECT 100.0 * (amount - prev_amount) / prev_amount)
    , 2) AS yoy_growth_pct
-- Cleaner null-safe division: use NULLIF to turn zero prev into NULL
FROM lagged
ORDER BY product, year;

-- Actually the cleanest null-safe expression:
SELECT
    product, year, amount, prev_amount,
    ROUND(
        100.0 * (amount - prev_amount) / NULLIF(prev_amount, 0),
        2
    ) AS yoy_growth_pct
FROM lagged
ORDER BY product, year;
```

**Follow-up questions the interviewer will ask:**

1. **"What does `NULLIF(prev_amount, 0)` do and why is it useful here?"**
   - `NULLIF(a, b)` returns NULL if `a = b`, otherwise returns `a`. So `NULLIF(prev_amount, 0)` turns a zero denominator into NULL. Division by NULL yields NULL (not an error), which is the correct output for "we can't compute growth from a zero baseline."

2. **"How would you also include quarter-over-quarter growth?"**
   - Add `quarter` to the table, change the `PARTITION BY` to include `product`, and `ORDER BY product, year, quarter`. The LAG then looks back one quarter within each product's series.

**Common mistakes that get you rejected:**
- Not handling `NULL` for the first year — the result is a NULL growth percentage, which is correct, but if you try to compute `(amount - NULL) / NULL` you still get NULL, so it works; the error is when developers show 0% instead of NULL for the first period.
- Divide-by-zero when `prev_amount = 0` — always use `NULLIF`.

---

### Q18: 7-Day Moving Average
**Company:** PhonePe, Groww, Swiggy Analytics, Amazon  **Difficulty:** 🟡  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Given a `daily_orders` table with `order_date` and `order_count`, compute the 7-day moving average of `order_count` (current day plus the previous 6 days). Round to 2 decimal places.

**What interviewer is testing:** ROWS BETWEEN frame clause, understanding that "7-day moving average" means the current row plus 6 preceding rows (total window = 7), handling the warm-up period (first 6 days have fewer than 7 data points).

**Ideal Answer:**

```sql
CREATE TABLE daily_orders (
    order_date  DATE PRIMARY KEY,
    order_count INT
);

INSERT INTO daily_orders VALUES
    ('2024-01-01', 100),
    ('2024-01-02', 120),
    ('2024-01-03',  90),
    ('2024-01-04', 110),
    ('2024-01-05', 130),
    ('2024-01-06', 105),
    ('2024-01-07', 115),
    ('2024-01-08', 140),
    ('2024-01-09', 125);

SELECT
    order_date,
    order_count,
    ROUND(
        AVG(order_count) OVER (
            ORDER BY order_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
            -- "6 PRECEDING": include the 6 rows before current + current row = 7 rows total
        ),
        2
    ) AS moving_avg_7d,
    COUNT(*) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS days_in_window   -- shows how many days are actually in the average (< 7 at start)
FROM daily_orders
ORDER BY order_date;

/*
 order_date | order_count | moving_avg_7d | days_in_window
------------+-------------+---------------+----------------
 2024-01-01 |         100 |        100.00 |              1
 2024-01-02 |         120 |        110.00 |              2
 2024-01-03 |          90 |        103.33 |              3
 2024-01-04 |         110 |        105.00 |              4
 2024-01-05 |         130 |        110.00 |              5
 2024-01-06 |         105 |        109.17 |              6
 2024-01-07 |         115 |        110.00 |              7  ← first full 7-day window
 2024-01-08 |         140 |        115.71 |              7
 2024-01-09 |         125 |        117.14 |              7
*/

-- Warm-up period: the first 6 rows have fewer than 7 data points.
-- AVG automatically uses however many rows are in the frame — this is correct behavior.
-- If the business requires ONLY full 7-day windows, add a filter:
-- WHERE (row_num >= 7) or use a CTE with ROW_NUMBER.

-- ─────────────────────────────────────────────
-- If you want ONLY rows with a full 7-day window:
-- ─────────────────────────────────────────────
SELECT order_date, order_count, moving_avg_7d
FROM (
    SELECT
        order_date,
        order_count,
        ROUND(AVG(order_count) OVER (
            ORDER BY order_date
            ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
        ), 2) AS moving_avg_7d,
        ROW_NUMBER() OVER (ORDER BY order_date) AS rn
    FROM daily_orders
) t
WHERE rn >= 7;
```

**Follow-up questions the interviewer will ask:**

1. **"What is the difference between `ROWS BETWEEN 6 PRECEDING AND CURRENT ROW` and `ROWS BETWEEN 6 PRECEDING AND 6 FOLLOWING`?"**
   - `6 PRECEDING AND CURRENT ROW`: trailing window (past 7 days including today). Used for moving averages in time-series analysis.
   - `6 PRECEDING AND 6 FOLLOWING`: centered window (±6 days around current). Total of 13 rows. Used in signal smoothing where future values are available.

2. **"What if there are gaps in dates (no data on some days)?"**
   - `ROWS BETWEEN 6 PRECEDING` counts 6 physical rows, not calendar days. If Jan 3 is missing, the Jan 7 row's window still includes Jan 1, 2, 4, 5, 6, 7 — only 6 rows, not truly 7 calendar days. For a true calendar-based 7-day window, use a date spine (Q33) to fill gaps, then apply the moving average.

**Common mistakes that get you rejected:**
- Using `ROWS BETWEEN 7 PRECEDING AND CURRENT ROW` — this creates an 8-row window (7 preceding + current = 8). Use 6 PRECEDING for a 7-day window.
- Using `RANGE BETWEEN` with a date-based frame instead of `ROWS BETWEEN` — `RANGE BETWEEN INTERVAL '6 days' PRECEDING AND CURRENT ROW` is the correct syntax for a calendar-based range in PostgreSQL, but requires the ORDER BY column to be a DATE/TIMESTAMP type.

---

### Q19: Pivot Rows to Columns (Static Pivot)
**Company:** Walmart Labs, Target, Flipkart Analytics  **Difficulty:** 🟡  **Frequency:** 🔥  **Round:** Onsite

**Question:** Given a `quarterly_sales` table with `year`, `quarter` (Q1–Q4), and `amount`, pivot the data so each quarter becomes a column. Explain why dynamic pivot is not natively supported in PostgreSQL.

**What interviewer is testing:** CASE-based aggregation as pivot technique, understanding of PostgreSQL's lack of native PIVOT syntax, and awareness of dynamic alternatives.

**Ideal Answer:**

```sql
CREATE TABLE quarterly_sales (
    year    INT,
    quarter VARCHAR(2),
    amount  NUMERIC
);

INSERT INTO quarterly_sales VALUES
    (2023, 'Q1', 100000), (2023, 'Q2', 120000),
    (2023, 'Q3', 115000), (2023, 'Q4', 130000),
    (2024, 'Q1', 110000), (2024, 'Q2', 135000),
    (2024, 'Q3', 140000), (2024, 'Q4', 150000);

-- ─────────────────────────────────────────────
-- Static pivot: CASE + GROUP BY
-- ─────────────────────────────────────────────
SELECT
    year,
    SUM(CASE WHEN quarter = 'Q1' THEN amount ELSE 0 END) AS q1,
    SUM(CASE WHEN quarter = 'Q2' THEN amount ELSE 0 END) AS q2,
    SUM(CASE WHEN quarter = 'Q3' THEN amount ELSE 0 END) AS q3,
    SUM(CASE WHEN quarter = 'Q4' THEN amount ELSE 0 END) AS q4,
    SUM(amount) AS total
FROM quarterly_sales
GROUP BY year
ORDER BY year;

/*
 year |    q1    |    q2    |    q3    |    q4    |  total
------+----------+----------+----------+----------+---------
 2023 |   100000 |   120000 |   115000 |   130000 |  465000
 2024 |   110000 |   135000 |   140000 |   150000 |  535000
*/

-- ─────────────────────────────────────────────
-- Why PostgreSQL has no native dynamic PIVOT
-- ─────────────────────────────────────────────
-- SQL is statically typed: the number and names of result columns must be
-- known at parse time, before the query executes. Dynamic column generation
-- (where column names come from data values) violates this constraint.
--
-- SQL Server has a PIVOT keyword; Oracle has PIVOT; MySQL requires the same
-- CASE technique. PostgreSQL offers no PIVOT keyword.
--
-- Workarounds for dynamic pivot in PostgreSQL:
-- 1. Use the `tablefunc` extension: crosstab() function — but column names
--    must still be specified in the query.
-- 2. Build the query string dynamically in PL/pgSQL using EXECUTE.
-- 3. Do the pivot in the application layer (Pandas, dplyr, etc.).
--    This is often the most practical approach for reporting.

-- ─────────────────────────────────────────────
-- crosstab() approach (requires: CREATE EXTENSION tablefunc)
-- ─────────────────────────────────────────────
CREATE EXTENSION IF NOT EXISTS tablefunc;

SELECT * FROM crosstab(
    'SELECT year, quarter, amount FROM quarterly_sales ORDER BY 1, 2',
    'SELECT DISTINCT quarter FROM quarterly_sales ORDER BY 1'
) AS ct(year INT, "Q1" NUMERIC, "Q2" NUMERIC, "Q3" NUMERIC, "Q4" NUMERIC);
-- The column list must be hardcoded — still static at query-parse time.
```

**Follow-up questions the interviewer will ask:**

1. **"How would you unpivot (columns to rows)?"**
   ```sql
   SELECT year, 'Q1' AS quarter, q1 AS amount FROM pivot_result
   UNION ALL
   SELECT year, 'Q2', q2 FROM pivot_result
   UNION ALL
   SELECT year, 'Q3', q3 FROM pivot_result
   UNION ALL
   SELECT year, 'Q4', q4 FROM pivot_result;
   ```
   Or in PostgreSQL using `VALUES` with a lateral join.

2. **"Why do you use SUM instead of MAX in the CASE pivot?"**
   - If only one row per year/quarter exists, both give the same result. `SUM` is safer: if there are multiple rows for the same year/quarter (duplicate data), `SUM` aggregates them correctly while `MAX` silently picks one value.

**Common mistakes that get you rejected:**
- Using `MAX(CASE ...)` which silently handles multiple rows incorrectly.
- Not explaining why dynamic pivot requires dynamic SQL or an application layer — interviewers specifically want to hear this limitation.

---

### Q20: Gap and Island Detection
**Company:** Swiggy, CRED, Meesho, Ola  **Difficulty:** 🔴  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Given a `subscriptions` table with `user_id`, `start_date`, and `end_date` for subscription periods, merge overlapping or adjacent intervals per user into consolidated ranges. This is the "merge intervals" problem in SQL.

**What interviewer is testing:** Gap-and-island technique using the ROW_NUMBER subtraction trick, date range merging logic, and CTE chaining.

**Ideal Answer:**

```sql
CREATE TABLE subscriptions (
    id         SERIAL PRIMARY KEY,
    user_id    INT,
    start_date DATE,
    end_date   DATE
);

INSERT INTO subscriptions (user_id, start_date, end_date) VALUES
    (1, '2024-01-01', '2024-01-10'),
    (1, '2024-01-08', '2024-01-15'),   -- overlaps with previous
    (1, '2024-01-16', '2024-01-20'),   -- adjacent (gap = 1 day, business-continuous)
    (1, '2024-02-01', '2024-02-10'),   -- separate island
    (2, '2024-01-05', '2024-01-12'),
    (2, '2024-01-20', '2024-01-25');

-- ─────────────────────────────────────────────
-- Step-by-step island merging
-- ─────────────────────────────────────────────

-- Step 1: For each row, find the maximum end_date seen so far in the partition.
-- If the current start_date > previous max end_date, it's a new island.
WITH ordered AS (
    SELECT
        user_id, start_date, end_date,
        -- The "island key" trick:
        -- MAX(end_date) of all prior rows (ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING)
        -- If current start_date > that max, this row starts a new island.
        MAX(end_date) OVER (
            PARTITION BY user_id
            ORDER BY start_date
            ROWS BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
        ) AS prev_max_end
    FROM subscriptions
),
flagged AS (
    SELECT
        user_id, start_date, end_date,
        -- Mark the start of each new island
        CASE
            WHEN prev_max_end IS NULL OR start_date > prev_max_end
            THEN 1 ELSE 0
        END AS is_new_island
    FROM ordered
),
island_num AS (
    SELECT
        user_id, start_date, end_date,
        SUM(is_new_island) OVER (
            PARTITION BY user_id
            ORDER BY start_date
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS island_id
    FROM flagged
)
SELECT
    user_id,
    island_id,
    MIN(start_date) AS merged_start,
    MAX(end_date)   AS merged_end,
    MAX(end_date) - MIN(start_date) + 1 AS duration_days
FROM island_num
GROUP BY user_id, island_id
ORDER BY user_id, merged_start;

/*
Expected output:
 user_id | island_id | merged_start | merged_end | duration_days
---------+-----------+--------------+------------+---------------
       1 |         1 | 2024-01-01   | 2024-01-20 |            20
       1 |         2 | 2024-02-01   | 2024-02-10 |            10
       2 |         1 | 2024-01-05   | 2024-01-12 |             8
       2 |         2 | 2024-01-20   | 2024-01-25 |             6
*/

-- Note: Jan 1-10 and Jan 8-15 overlap → merged to Jan 1-15.
-- Jan 16-20 is adjacent (one day gap) → if business treats gap=1 as continuous,
-- use: start_date > prev_max_end + INTERVAL '1 day' as the new-island condition.
-- With gap=0 threshold, it merges to Jan 1-20. ✓

-- ─────────────────────────────────────────────
-- Simpler "consecutive integers" variant (the classic gap-and-island)
-- ─────────────────────────────────────────────
-- For a sequence of integers (not date ranges), the trick is:
--   group_key = value - ROW_NUMBER() OVER (ORDER BY value)
-- Consecutive integers all produce the same group_key.

CREATE TABLE int_sequence (val INT);
INSERT INTO int_sequence VALUES (1),(2),(3),(5),(6),(7),(10);

SELECT
    MIN(val) AS island_start,
    MAX(val) AS island_end,
    COUNT(*) AS island_size
FROM (
    SELECT val,
           val - ROW_NUMBER() OVER (ORDER BY val) AS grp
    FROM int_sequence
) t
GROUP BY grp
ORDER BY island_start;

/*
 island_start | island_end | island_size
--------------+------------+-------------
            1 |          3 |           3
            5 |          7 |           3
           10 |         10 |           1
*/
```

**Follow-up questions the interviewer will ask:**

1. **"Why does `value - ROW_NUMBER()` create a constant group key for consecutive integers?"**
   - For consecutive values (1, 2, 3), the row numbers are also (1, 2, 3). The difference (1-1=0, 2-2=0, 3-3=0) is constant. When a gap appears (5 after 3), the value jumps by 2 but the row number jumps by 1, so the difference changes (5-4=1), marking a new island.

2. **"How would you find gaps (the missing ranges) rather than islands?"**
   ```sql
   SELECT island_end + 1 AS gap_start,
          LEAD(island_start) OVER (ORDER BY island_start) - 1 AS gap_end
   FROM (
       SELECT MIN(val) AS island_start, MAX(val) AS island_end
       FROM (SELECT val, val - ROW_NUMBER() OVER (ORDER BY val) AS grp FROM int_sequence) t
       GROUP BY grp
   ) islands
   WHERE LEAD(island_start) OVER (ORDER BY island_start) IS NOT NULL;
   ```

3. **"How does this relate to the 'Merge Intervals' LeetCode problem?"**
   - It is the SQL equivalent. The date-range merging CTE above is the direct SQL translation of the classic merge-intervals algorithm (sort by start, extend end greedily).

**Common mistakes that get you rejected:**
- Using `ROW_NUMBER` on the raw sequence without ordering by the value — produces arbitrary groupings.
- For date ranges: using the simple `date - ROW_NUMBER()` trick directly (works for consecutive days but breaks for overlapping intervals, which require the MAX end_date approach shown above).
- Not handling overlapping intervals (just merging adjacent but not overlapping) — missing the `MAX(end_date) ... 1 PRECEDING` step.

---

### Q21: Mutual Friends — Self-Join on Friendship Table
**Company:** Facebook/Meta, LinkedIn  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** You have a `friends` table with columns `(user1_id, user2_id)`. Each row means user1 and user2 are friends. The friendship is stored once (not mirrored). Find all mutual friends between user 1 and user 2 — that is, users who are friends with both user 1 and user 2.

**What interviewer is testing:** Whether you can handle a symmetric relationship stored asymmetrically, write a self-join on the same table, and know when to use `INTERSECT` as a cleaner alternative.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE friends (
    user1_id INT,
    user2_id INT,
    PRIMARY KEY (user1_id, user2_id),
    CHECK (user1_id < user2_id)   -- enforce canonical order so no duplicates
);

INSERT INTO friends VALUES
(1, 2),
(1, 3),
(1, 4),
(2, 3),
(2, 5),
(3, 5),
(4, 5);
-- User 1's friends: 2, 3, 4
-- User 2's friends: 1, 3, 5
-- Mutual friends of 1 and 2: user 3
```

**Step 1 — create a symmetric view so direction doesn't matter:**

Because rows are stored with `user1_id < user2_id`, a user who is "friend B" never appears in `user1_id`. We need a union to make the table symmetric:

```sql
CREATE VIEW all_friendships AS
    SELECT user1_id AS person, user2_id AS friend FROM friends
    UNION ALL
    SELECT user2_id AS person, user1_id AS friend FROM friends;
```

**Approach A — Self-Join:**
```sql
SELECT f1.friend AS mutual_friend
FROM   all_friendships f1
JOIN   all_friendships f2
       ON f1.friend = f2.friend
WHERE  f1.person = 1
AND    f2.person = 2;

-- Result:
-- mutual_friend
-- -------------
-- 3
```

How it works: `f1` gives all friends of user 1, `f2` gives all friends of user 2, joining on `friend` gives the intersection.

**Approach B — INTERSECT (cleaner):**
```sql
SELECT friend FROM all_friendships WHERE person = 1
INTERSECT
SELECT friend FROM all_friendships WHERE person = 2;

-- Result: 3
```

`INTERSECT` automatically deduplicates, so it reads more cleanly and avoids accidental duplicates if the friendship data is messy.

**Generalised: mutual friends of any pair (parameterised):**
```sql
-- Replace 1 and 2 with $1 and $2 in a prepared statement / function
SELECT friend
FROM   all_friendships
WHERE  person = $1
INTERSECT
SELECT friend
FROM   all_friendships
WHERE  person = $2;
```

**Follow-up questions the interviewer will ask:**

1. **"What if the friendship table already stores both directions (1→2 and 2→1)?"**
   Drop the UNION ALL view — query directly. INTERSECT still works perfectly. The self-join works too but you must add `AND f1.friend != 1 AND f1.friend != 2` to exclude the query users themselves if they appear as friends of each other.

2. **"How do you scale this to find mutual friend count for ALL pairs?"**
   ```sql
   SELECT f1.person AS user_a, f2.person AS user_b, COUNT(*) AS mutual_count
   FROM   all_friendships f1
   JOIN   all_friendships f2 ON f1.friend = f2.friend
   WHERE  f1.person < f2.person   -- avoid counting (1,2) and (2,1) separately
   GROUP  BY f1.person, f2.person;
   ```
   At Facebook scale this becomes a graph problem solved with distributed joins (Spark, Presto), not a single SQL query.

3. **"When does INTERSECT fail?"**
   If the two SELECT lists have different columns or types the query errors. Also `INTERSECT` uses `INTERSECT ALL` to keep duplicates — use `INTERSECT` (not `INTERSECT ALL`) when you want distinct results.

**Common mistakes that get you rejected:**
- Querying the raw `friends` table without making it symmetric — misses all cases where the mutual friend is stored as `user2_id`.
- Using `IN` with a subquery instead of INTERSECT: `WHERE person = 1 AND friend IN (SELECT friend FROM all_friendships WHERE person = 2)` — works but is O(n²) without an index.
- Forgetting to exclude the two query users themselves if they are directly friends (user 1 and user 2 would appear as each other's "mutual friend" if you're not careful).
- Using `UNION` instead of `UNION ALL` in the symmetric view — `UNION` deduplicates and is slower; each edge appears exactly once in each direction so there are no actual duplicates to remove.

---

### Q22: User Retention Cohort — Day-1 / Day-7 / Day-30 Retention
**Company:** Flipkart, Swiggy, Meesho  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** You have a `user_events` table recording every user login. Calculate Day-1, Day-7, and Day-30 retention: for each weekly signup cohort, what percentage of users came back exactly (or within) 1 day, 7 days, and 30 days after their first login?

**What interviewer is testing:** DATE_TRUNC for cohort bucketing, self-join on the same user with date arithmetic, aggregate + conditional COUNT for multi-column pivot in one pass, and understanding of "exact day" vs "within N days" retention definitions.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE user_events (
    user_id    INT,
    event_date DATE
);

INSERT INTO user_events VALUES
(1, '2024-01-01'), (1, '2024-01-02'), (1, '2024-01-08'), (1, '2024-01-31'),
(2, '2024-01-01'), (2, '2024-01-03'),
(3, '2024-01-01'), (3, '2024-01-08'),
(4, '2024-01-07'), (4, '2024-01-08'), (4, '2024-01-14'),
(5, '2024-01-07'),
(6, '2024-01-14'), (6, '2024-01-15'), (6, '2024-01-21'), (6, '2024-02-13');
```

**Step 1 — find each user's first login (cohort date):**
```sql
WITH first_login AS (
    SELECT user_id,
           MIN(event_date)                          AS cohort_date,
           DATE_TRUNC('week', MIN(event_date))::DATE AS cohort_week
    FROM   user_events
    GROUP  BY user_id
),
```

**Step 2 — join back to all events to measure gap:**
```sql
activity AS (
    SELECT fl.user_id,
           fl.cohort_week,
           ue.event_date - fl.cohort_date AS days_since_first
    FROM   first_login fl
    JOIN   user_events ue ON fl.user_id = ue.user_id
    WHERE  ue.event_date > fl.cohort_date   -- exclude the first login itself
),
```

**Step 3 — flag retention windows (within means ≤ N days):**
```sql
retention_flags AS (
    SELECT cohort_week,
           user_id,
           MAX(CASE WHEN days_since_first BETWEEN 1  AND  1  THEN 1 ELSE 0 END) AS retained_d1,
           MAX(CASE WHEN days_since_first BETWEEN 1  AND  7  THEN 1 ELSE 0 END) AS retained_d7,
           MAX(CASE WHEN days_since_first BETWEEN 1  AND 30  THEN 1 ELSE 0 END) AS retained_d30
    FROM   activity
    GROUP  BY cohort_week, user_id
)
```

**Step 4 — aggregate per cohort:**
```sql
SELECT
    cw.cohort_week,
    COUNT(DISTINCT cw.user_id)                          AS cohort_size,
    ROUND(100.0 * SUM(COALESCE(rf.retained_d1,  0)) / COUNT(DISTINCT cw.user_id), 1) AS d1_pct,
    ROUND(100.0 * SUM(COALESCE(rf.retained_d7,  0)) / COUNT(DISTINCT cw.user_id), 1) AS d7_pct,
    ROUND(100.0 * SUM(COALESCE(rf.retained_d30, 0)) / COUNT(DISTINCT cw.user_id), 1) AS d30_pct
FROM (SELECT DISTINCT cohort_week, user_id FROM first_login) cw
LEFT JOIN retention_flags rf
       ON cw.cohort_week = rf.cohort_week AND cw.user_id = rf.user_id
GROUP  BY cw.cohort_week
ORDER  BY cw.cohort_week;

-- Expected output (approximate):
-- cohort_week | cohort_size | d1_pct | d7_pct | d30_pct
-- ------------+-------------+--------+--------+--------
-- 2024-01-01  |      4      |  50.0  |  75.0  | 100.0
-- 2024-01-07  |      2      |  50.0  | 100.0  |  50.0
-- 2024-01-14  |      1      | 100.0  | 100.0  | 100.0
```

**Key design decisions explained:**
- `LEFT JOIN` in step 4: users who never came back still count in the denominator.
- `MAX(CASE WHEN ...)` per user: a user who visited on day 3 still counts as D7-retained (within 7 days).
- `COALESCE(rf.retained_d1, 0)`: users with no return visits produce a NULL from the LEFT JOIN — treat as 0.
- `DATE_TRUNC('week', ...)`: groups cohorts by week; change to `'month'` for monthly cohorts.

**Follow-up questions the interviewer will ask:**

1. **"How would you compute N-day rolling retention (not cumulative)?"**
   Change `BETWEEN 1 AND 7` to `BETWEEN 6 AND 8` (±1 day window around day 7). "Exact" D7 retention is typically defined as returning between day 6 and day 8 to account for timezone drift.

2. **"How do you handle users who signed up today — they haven't had 30 days yet?"**
   Add a filter: `WHERE fl.cohort_date <= CURRENT_DATE - INTERVAL '30 days'` before computing D30 retention. Otherwise the denominator for D30 includes users who couldn't possibly have hit that window yet, understating retention.

3. **"What's the difference between retention and return rate?"**
   Retention = came back on/by day N. Return rate = average number of return visits per user. Retention is a binary flag; return rate is continuous.

**Common mistakes that get you rejected:**
- Forgetting the `LEFT JOIN` — only users who returned appear in the result, making D1 retention always 100%.
- Using `COUNT(DISTINCT user_id)` on the joined table instead of on the cohort CTE — users with multiple return visits get double-counted.
- Defining "Day 1" as `event_date = cohort_date + 1` (exact) rather than `<= cohort_date + 1` (within) without clarifying the definition with the interviewer — the right answer depends on the business definition.
- `DATE_TRUNC` returns a `TIMESTAMP` in PostgreSQL — cast to `::DATE` or comparisons against `DATE` columns silently fail.

---

### Q23: Recursive CTE — Full Org Chart at Any Depth
**Company:** Amazon, Google, Uber  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** You have an `employees` table with columns `(id, name, manager_id)`. `manager_id` is NULL for the CEO. Write a query to find all employees who report — directly or indirectly — to the manager with `id = 5`. Include each employee's depth in the hierarchy and their full reporting path. Explain the termination condition, anchor vs recursive member, and how to detect cycles.

**What interviewer is testing:** Mastery of recursive CTEs (the hardest standard SQL feature), ability to explain the execution model, awareness of cycle detection, and ability to augment the query with depth tracking and path strings.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE employees (
    id         INT PRIMARY KEY,
    name       VARCHAR(50),
    manager_id INT REFERENCES employees(id)
);

INSERT INTO employees VALUES
(1,  'Alice',   NULL),   -- CEO
(2,  'Bob',     1),      -- reports to Alice
(3,  'Carol',   1),      -- reports to Alice
(4,  'Dave',    2),      -- reports to Bob
(5,  'Eve',     2),      -- reports to Bob  ← our target manager
(6,  'Frank',   5),      -- reports to Eve
(7,  'Grace',   5),      -- reports to Eve
(8,  'Heidi',   6),      -- reports to Frank
(9,  'Ivan',    6),      -- reports to Frank
(10, 'Judy',    7),      -- reports to Grace
(11, 'Mallory', 3);      -- reports to Carol (different branch)
```

**Full recursive CTE with depth and path:**
```sql
WITH RECURSIVE org_chart AS (

    -- ── ANCHOR MEMBER ──────────────────────────────────────────────
    -- Start with the target manager (id = 5). This is the base case.
    SELECT
        id,
        name,
        manager_id,
        0              AS depth,
        ARRAY[id]      AS path,        -- track visited IDs for cycle detection
        id::TEXT       AS path_names   -- human-readable path string
    FROM employees
    WHERE id = 5

    UNION ALL

    -- ── RECURSIVE MEMBER ───────────────────────────────────────────
    -- Join the employees table to the current result set.
    -- Each iteration goes one level deeper.
    SELECT
        e.id,
        e.name,
        e.manager_id,
        oc.depth + 1,
        oc.path || e.id,                           -- append new ID to path array
        oc.path_names || ' → ' || e.name           -- append name to path string
    FROM employees e
    JOIN org_chart oc ON e.manager_id = oc.id      -- child's manager = current node
    WHERE NOT e.id = ANY(oc.path)                  -- ← CYCLE DETECTION
)

SELECT
    id,
    name,
    depth,
    path_names AS reporting_path
FROM org_chart
WHERE id != 5        -- exclude the manager themselves if you want only reports
ORDER BY depth, id;

-- Expected output:
-- id | name  | depth | reporting_path
-- ---+-------+-------+-------------------------------
--  6 | Frank |   1   | 5 → Frank
--  7 | Grace |   1   | 5 → Grace
--  8 | Heidi |   2   | 5 → Frank → Heidi
--  9 | Ivan  |   2   | 5 → Frank → Ivan
-- 10 | Judy  |   2   | 5 → Grace → Judy
```

**How recursive CTEs execute — the mental model:**

```
Iteration 0 (anchor):  result = { Eve(5, depth=0) }
Iteration 1:           find employees where manager_id IN {5}   → Frank(6), Grace(7)
                       result += { Frank(depth=1), Grace(depth=1) }
Iteration 2:           find employees where manager_id IN {6,7} → Heidi(8), Ivan(9), Judy(10)
                       result += { Heidi(depth=2), Ivan(depth=2), Judy(depth=2) }
Iteration 3:           find employees where manager_id IN {8,9,10} → nobody
                       result += {} → EMPTY SET → recursion terminates
```

**Termination condition:** Recursion stops when the recursive member produces zero rows. In a well-formed org chart with no cycles this happens naturally when leaf nodes (employees with no direct reports) are reached.

**Anchor member vs recursive member:**
- **Anchor member**: the non-recursive SELECT that provides the initial seed rows. Executed once.
- **Recursive member**: the SELECT that references the CTE name itself (`org_chart`). Executed repeatedly, each time operating on only the *new rows added in the previous iteration* (not the full accumulated result set).
- They must be joined with `UNION ALL` (or `UNION`). `UNION ALL` is almost always correct — `UNION` deduplicates every iteration which is expensive and hides bugs.

**Cycle detection explained:**
```sql
-- Without cycle detection, a bad data row like:
INSERT INTO employees VALUES (99, 'Bad', 99);  -- reports to itself
-- ...would cause infinite recursion and an error:
-- ERROR: infinite recursion detected in recursive query "org_chart"

-- The fix: track visited IDs in an ARRAY and stop if we see a repeat:
WHERE NOT e.id = ANY(oc.path)
-- PostgreSQL also has a built-in safeguard: max_recursion_depth (default 100)
-- which raises an error before the session hangs.
```

**Finding the manager of a node (bottom-up traversal):**
```sql
-- Reverse: start from an employee and walk UP to the CEO
WITH RECURSIVE chain AS (
    SELECT id, name, manager_id, 0 AS depth
    FROM   employees WHERE id = 9              -- start: Ivan

    UNION ALL

    SELECT e.id, e.name, e.manager_id, c.depth + 1
    FROM   employees e
    JOIN   chain c ON e.id = c.manager_id      -- parent's ID = current manager_id
)
SELECT * FROM chain ORDER BY depth;
-- Returns: Ivan → Frank → Eve → Bob → Alice
```

**Follow-up questions the interviewer will ask:**

1. **"What happens if you use UNION instead of UNION ALL in the recursive member?"**
   PostgreSQL deduplicates after every iteration. For an org chart with no repeated employees this produces the same result but is significantly slower. More importantly, it can mask cycles — a cycle would just stop adding duplicates rather than being detected. Always prefer `UNION ALL` and handle deduplication explicitly if needed.

2. **"How would you find the depth of the entire org chart (the longest path from CEO to any leaf)?"**
   ```sql
   WITH RECURSIVE full_chart AS (
       SELECT id, 0 AS depth FROM employees WHERE manager_id IS NULL
       UNION ALL
       SELECT e.id, fc.depth + 1
       FROM   employees e JOIN full_chart fc ON e.manager_id = fc.id
   )
   SELECT MAX(depth) AS org_depth FROM full_chart;
   ```

3. **"Is this query supported in MySQL?"**
   Yes — `WITH RECURSIVE` is supported in MySQL 8.0+, PostgreSQL 8.4+, SQL Server (as `WITH ... AS`), SQLite 3.8.3+. Not supported in MySQL 5.x.

**Common mistakes that get you rejected:**
- Writing `UNION` instead of `UNION ALL` and not explaining the difference.
- Forgetting the anchor member — the query won't compile.
- Not including cycle detection when the interviewer asks about robustness.
- Self-referencing the CTE in the anchor member — only the recursive member may reference the CTE name.
- Confusing "recursive member operates on all accumulated rows" (wrong) vs "operates only on rows added in the previous iteration" (correct).

---

### Q24: Session Identification from Events — LAG + Cumulative SUM
**Company:** Hotstar, Netflix, Flipkart  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** You have a `page_views` table with columns `(user_id, page, viewed_at TIMESTAMPTZ)`. A new session starts when a user's gap between consecutive events is greater than 30 minutes. Assign a session ID to every event. Find the total number of sessions per user.

**What interviewer is testing:** LAG window function, EXTRACT(EPOCH) for interval-to-seconds conversion, the SUM-of-flags pattern to assign group IDs (a reusable pattern for any "island detection" problem), and PARTITION BY ordering.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE page_views (
    user_id   INT,
    page      VARCHAR(50),
    viewed_at TIMESTAMPTZ
);

INSERT INTO page_views VALUES
(1, '/home',    '2024-01-01 10:00:00+00'),
(1, '/about',   '2024-01-01 10:05:00+00'),
(1, '/contact', '2024-01-01 10:10:00+00'),  -- same session (5 min gap)
(1, '/home',    '2024-01-01 11:00:00+00'),  -- NEW session (50 min gap > 30)
(1, '/pricing', '2024-01-01 11:15:00+00'),  -- same session
(2, '/home',    '2024-01-01 09:00:00+00'),
(2, '/docs',    '2024-01-01 09:20:00+00'),  -- same session
(2, '/home',    '2024-01-01 12:00:00+00'),  -- NEW session (160 min gap)
(2, '/signup',  '2024-01-01 12:05:00+00');  -- same session
```

**Step 1 — use LAG to get the previous event timestamp:**
```sql
WITH lag_step AS (
    SELECT
        user_id,
        page,
        viewed_at,
        LAG(viewed_at) OVER (
            PARTITION BY user_id
            ORDER BY viewed_at
        ) AS prev_viewed_at
    FROM page_views
),
```

**Step 2 — flag the start of a new session:**
```sql
gap_step AS (
    SELECT
        user_id,
        page,
        viewed_at,
        prev_viewed_at,
        EXTRACT(EPOCH FROM (viewed_at - prev_viewed_at)) AS gap_seconds,
        CASE
            WHEN prev_viewed_at IS NULL THEN 1         -- first event ever = new session
            WHEN EXTRACT(EPOCH FROM (viewed_at - prev_viewed_at)) > 1800
                 THEN 1                                -- gap > 30 min = new session
            ELSE 0
        END AS is_new_session
    FROM lag_step
),
```

**Step 3 — cumulative SUM of new-session flags = session ID:**
```sql
session_step AS (
    SELECT
        user_id,
        page,
        viewed_at,
        gap_seconds,
        is_new_session,
        SUM(is_new_session) OVER (
            PARTITION BY user_id
            ORDER BY viewed_at
            ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
        ) AS session_id
    FROM gap_step
)

SELECT * FROM session_step ORDER BY user_id, viewed_at;

-- Expected output:
-- user_id | page      | viewed_at            | gap_seconds | is_new_session | session_id
-- --------+-----------+----------------------+-------------+----------------+-----------
--    1    | /home     | 2024-01-01 10:00 UTC |    NULL     |       1        |     1
--    1    | /about    | 2024-01-01 10:05 UTC |     300     |       0        |     1
--    1    | /contact  | 2024-01-01 10:10 UTC |     300     |       0        |     1
--    1    | /home     | 2024-01-01 11:00 UTC |    3000     |       1        |     2
--    1    | /pricing  | 2024-01-01 11:15 UTC |     900     |       0        |     2
--    2    | /home     | 2024-01-01 09:00 UTC |    NULL     |       1        |     1
--    2    | /docs     | 2024-01-01 09:20 UTC |    1200     |       0        |     1
--    2    | /home     | 2024-01-01 12:00 UTC |    9600     |       1        |     2
--    2    | /signup   | 2024-01-01 12:05 UTC |     300     |       0        |     2
```

**Final aggregation — sessions per user:**
```sql
SELECT user_id, MAX(session_id) AS total_sessions
FROM   session_step
GROUP  BY user_id;

-- user_id | total_sessions
-- --------+---------------
--    1    |      2
--    2    |      2
```

**Why `SUM(is_new_session) OVER (...)` assigns session IDs:**
Every time `is_new_session = 1`, the running sum increments by 1. Events within the same session have `is_new_session = 0` so the sum stays flat. The first event of each user always gets flag = 1, so the first session is always session ID 1 (not 0).

**Follow-up questions the interviewer will ask:**

1. **"Why use `EXTRACT(EPOCH FROM ...)` instead of comparing timestamps directly?"**
   Timestamp subtraction in PostgreSQL produces an `INTERVAL`, not a number. You cannot compare an `INTERVAL > 1800`. `EXTRACT(EPOCH FROM interval)` converts it to total seconds as a float, making numeric comparison straightforward. Alternatively: `viewed_at - prev_viewed_at > INTERVAL '30 minutes'` — this works in PostgreSQL and is more readable, but EPOCH is portable and easier to tune (just change the number).

2. **"How do you create a globally unique session ID across users?"**
   `user_id::TEXT || '-' || session_id::TEXT` or use a composite key `(user_id, session_id)`. For a true UUID: `MD5(user_id::TEXT || session_id::TEXT)` (deterministic) or generate a surrogate key in a separate step.

3. **"What if events are not in order in the table? Does LAG still work?"**
   Yes — `LAG(...) OVER (PARTITION BY user_id ORDER BY viewed_at)` always operates on the logical sort order, not physical storage order. The `ORDER BY` inside the window frame is mandatory here.

**Common mistakes that get you rejected:**
- Forgetting `PARTITION BY user_id` — LAG spans across different users, assigning incorrect gaps.
- Comparing `INTERVAL > 1800` directly — PostgreSQL will error; you must extract epoch or compare to `INTERVAL '30 minutes'`.
- Not handling `prev_viewed_at IS NULL` (the very first event per user) — without the NULL check, the first event gets `is_new_session = 0` and session ID 0.
- Using `RANGE BETWEEN` instead of `ROWS BETWEEN` in the SUM — `RANGE` with an ORDER BY on a timestamp is fine here but can produce unexpected ties; `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW` is explicit and correct.

---

### Q25: Top-N Per Group Without Window Functions — Correlated Subquery
**Company:** Infosys, TCS, Wipro  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥  **Round:** OA

**Question:** Find the top 3 highest-paid employees in each department. Show both the correlated subquery approach (without window functions) and the window function approach. Explain the performance difference.

**What interviewer is testing:** Understanding of correlated subqueries, the COUNT(*) < N trick, and why window functions exist — ability to compare approaches analytically.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE employees (
    id         INT PRIMARY KEY,
    name       VARCHAR(50),
    department VARCHAR(30),
    salary     INT
);

INSERT INTO employees VALUES
(1, 'Alice',   'Engineering', 120000),
(2, 'Bob',     'Engineering', 110000),
(3, 'Carol',   'Engineering', 105000),
(4, 'Dave',    'Engineering',  95000),
(5, 'Eve',     'Marketing',    90000),
(6, 'Frank',   'Marketing',    85000),
(7, 'Grace',   'Marketing',    80000),
(8, 'Heidi',   'Marketing',    75000),
(9, 'Ivan',    'HR',           70000),
(10,'Judy',    'HR',           65000);
```

**Approach A — Correlated Subquery (no window functions):**
```sql
SELECT e1.name, e1.department, e1.salary
FROM   employees e1
WHERE (
    -- Count how many employees in the same dept earn MORE than e1
    SELECT COUNT(*)
    FROM   employees e2
    WHERE  e2.department = e1.department
    AND    e2.salary > e1.salary
) < 3                        -- fewer than 3 earn more → e1 is top-3
ORDER  BY e1.department, e1.salary DESC;

-- Expected output:
-- name  | department  | salary
-- ------+-------------+--------
-- Alice | Engineering | 120000
-- Bob   | Engineering | 110000
-- Carol | Engineering | 105000
-- Eve   | Marketing   |  90000
-- Frank | Marketing   |  85000
-- Grace | Marketing   |  80000
-- Ivan  | HR          |  70000
-- Judy  | HR          |  65000
```

**How it works:** For each row `e1`, the subquery counts colleagues in the same department earning strictly more. If fewer than 3 earn more, `e1` must be in the top 3. The anchor is `< N` where N is the rank cutoff.

**Tie-handling caveat:** If two employees share the same salary at the boundary, both pass the filter. This mirrors `RANK()` behaviour (both rank 3, next is rank 5). To mimic `DENSE_RANK`, use `<= 3` — but that can return more than N rows. To mimic `ROW_NUMBER`, add a secondary tie-break in both the subquery and the outer WHERE.

**Approach B — Window Function (correct, fast):**
```sql
SELECT name, department, salary
FROM (
    SELECT
        name, department, salary,
        DENSE_RANK() OVER (
            PARTITION BY department
            ORDER BY salary DESC
        ) AS rnk
    FROM employees
) ranked
WHERE rnk <= 3
ORDER BY department, salary DESC;

-- Same output as above
```

**Performance comparison:**

| Approach | Time complexity | Why |
|---|---|---|
| Correlated subquery | O(n²) | For each of n rows, the inner query scans up to n rows of the same department |
| Window function | O(n log n) | Single pass with a sort; database uses a hash or sort aggregate internally |

For 1 million rows, the correlated subquery might take 30+ seconds; the window function takes under 1 second. The query optimiser can sometimes convert a correlated subquery to a hash join, but it's not guaranteed.

**Follow-up questions the interviewer will ask:**

1. **"What's the difference between ROW_NUMBER, RANK, and DENSE_RANK for this use case?"**
   - `ROW_NUMBER`: always returns exactly N rows, breaks ties arbitrarily.
   - `RANK`: tied employees share the same rank; the next rank skips (1,2,2,4).
   - `DENSE_RANK`: tied employees share rank, no gaps (1,2,2,3). Usually correct for "top N" — you don't want to exclude an employee just because a colleague shares their salary.

2. **"Can you optimise the correlated subquery with an index?"**
   An index on `(department, salary DESC)` helps the inner COUNT scan only the relevant department. This reduces practical time but the theoretical complexity is still O(n²) in the worst case (one department with all employees).

**Common mistakes that get you rejected:**
- Writing `< N` as `<= N` — that returns top N+1 (employees ranked 1 through N by COUNT, but rank 0 means #1 earner, rank 1 means #2 earner — off by one).
- Not handling ties: assuming exactly 3 rows come back when two employees tie for 3rd place.
- Forgetting `PARTITION BY department` in the window function — ranks globally, not per department.
- Using `ROW_NUMBER` when `DENSE_RANK` is more appropriate for salary rankings.

---

### Q26: First and Last Purchase Per Customer
**Company:** Amazon, Meesho, Myntra  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** From an `orders` table `(order_id, customer_id, amount, order_date)`, find for each customer: their first purchase date and amount, and their last purchase date and amount. Show both the MIN/MAX + GROUP BY approach and the FIRST_VALUE/LAST_VALUE window function approach. Explain the LAST_VALUE frame gotcha.

**What interviewer is testing:** Whether you know that `LAST_VALUE` has a non-obvious default window frame that breaks naive usage, and whether you can use self-joins or subqueries to retrieve attribute columns alongside MIN/MAX aggregates.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE orders (
    order_id    INT PRIMARY KEY,
    customer_id INT,
    amount      NUMERIC(10,2),
    order_date  DATE
);

INSERT INTO orders VALUES
(1, 101, 50.00,  '2024-01-05'),
(2, 101, 120.00, '2024-03-10'),
(3, 101, 30.00,  '2024-06-20'),
(4, 102, 200.00, '2024-02-01'),
(5, 102, 75.00,  '2024-05-15'),
(6, 103, 90.00,  '2024-04-01');
```

**Approach A — MIN/MAX with GROUP BY + self-join for amounts:**

```sql
WITH bounds AS (
    SELECT
        customer_id,
        MIN(order_date) AS first_date,
        MAX(order_date) AS last_date
    FROM orders
    GROUP BY customer_id
)
SELECT
    b.customer_id,
    b.first_date,
    o_first.amount  AS first_amount,
    b.last_date,
    o_last.amount   AS last_amount
FROM bounds b
JOIN orders o_first
     ON o_first.customer_id = b.customer_id
     AND o_first.order_date = b.first_date
JOIN orders o_last
     ON o_last.customer_id = b.customer_id
     AND o_last.order_date = b.last_date;

-- Expected output:
-- customer_id | first_date | first_amount | last_date  | last_amount
-- ------------+------------+--------------+------------+------------
--     101     | 2024-01-05 |    50.00     | 2024-06-20 |   30.00
--     102     | 2024-02-01 |   200.00     | 2024-05-15 |   75.00
--     103     | 2024-04-01 |    90.00     | 2024-04-01 |   90.00
```

**Approach B — FIRST_VALUE and LAST_VALUE (window functions):**

```sql
SELECT DISTINCT
    customer_id,
    FIRST_VALUE(order_date) OVER w                          AS first_date,
    FIRST_VALUE(amount)     OVER w                          AS first_amount,
    -- LAST_VALUE requires explicit frame extension!
    LAST_VALUE(order_date)  OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    )                                                       AS last_date,
    LAST_VALUE(amount)      OVER (
        PARTITION BY customer_id
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    )                                                       AS last_amount
FROM orders
WINDOW w AS (
    PARTITION BY customer_id
    ORDER BY order_date
    ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
);
```

**The LAST_VALUE frame gotcha — this is a classic interview trap:**

When you write:
```sql
LAST_VALUE(amount) OVER (PARTITION BY customer_id ORDER BY order_date)
```

The default window frame is `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This means for every row, `LAST_VALUE` returns the value of the *current row itself* — not the last row in the partition. It only gives the true last value for the last row in each partition.

The fix: always specify `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` when using `LAST_VALUE`.

**Simpler alternative — use FIRST_VALUE with DESC ordering:**
```sql
SELECT DISTINCT
    customer_id,
    FIRST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date ASC)  AS first_date,
    FIRST_VALUE(amount)     OVER (PARTITION BY customer_id ORDER BY order_date ASC)  AS first_amount,
    FIRST_VALUE(order_date) OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS last_date,
    FIRST_VALUE(amount)     OVER (PARTITION BY customer_id ORDER BY order_date DESC) AS last_amount
FROM orders;
-- FIRST_VALUE always uses UNBOUNDED PRECEDING to CURRENT ROW by default,
-- which correctly returns the first row of the ordered partition.
```

This avoids the LAST_VALUE frame problem entirely by reversing the sort direction.

**Follow-up questions the interviewer will ask:**

1. **"What happens with the MIN/MAX approach if two orders share the same date?"**
   The JOIN returns multiple rows for that customer. Add `ORDER BY order_id LIMIT 1` in a lateral subquery, or use `DISTINCT ON (customer_id)` in PostgreSQL to pick one deterministically.

2. **"Which approach is faster for large tables?"**
   The window function approach scans once; the MIN/MAX + self-join approach scans twice (once for bounds, twice for attribute lookup). With an index on `(customer_id, order_date)`, both are fast. At scale, the single-scan window approach wins.

**Common mistakes that get you rejected:**
- Using `LAST_VALUE` without specifying the frame — returns the current row's value, not the partition's last value. This is one of the most common SQL gotchas.
- Selecting `MIN(order_date), amount` directly in a GROUP BY query — `amount` is not in the GROUP BY and not aggregated; this is a SQL error in standard SQL (though MySQL with `ONLY_FULL_GROUP_BY` off would silently return an arbitrary value).
- Not accounting for ties on the boundary date with the JOIN approach.

---

### Q27: Median Per Department — Manual Calculation Without PERCENTILE_CONT
**Company:** Razorpay, CRED, Groww  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** You have a `salaries` table `(employee_id, department, salary)`. Find the median salary per department. Show the manual approach using ROW_NUMBER, and also show the PERCENTILE_CONT approach.

**What interviewer is testing:** Ability to implement statistics manually in SQL, understanding of odd vs even count median logic, and awareness of the built-in ordered-set aggregate.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE salaries (
    employee_id INT PRIMARY KEY,
    department  VARCHAR(30),
    salary      INT
);

INSERT INTO salaries VALUES
(1,  'Engineering', 120000),
(2,  'Engineering', 110000),
(3,  'Engineering', 105000),
(4,  'Engineering',  95000),
(5,  'Engineering',  80000),   -- 5 rows → median is 3rd = 105000
(6,  'Marketing',    90000),
(7,  'Marketing',    85000),
(8,  'Marketing',    80000),
(9,  'Marketing',    75000),   -- 4 rows → median is avg of 2nd+3rd = (85000+80000)/2 = 82500
(10, 'HR',           70000);   -- 1 row → median = 70000
```

**Approach A — Manual median using ROW_NUMBER:**

```sql
WITH ranked AS (
    SELECT
        department,
        salary,
        ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary) AS rn,
        COUNT(*)     OVER (PARTITION BY department)                 AS cnt
    FROM salaries
),
median_rows AS (
    SELECT department, salary, rn, cnt
    FROM   ranked
    WHERE
        -- For odd count: pick the middle row (rn = (cnt+1)/2)
        -- For even count: pick the two middle rows (rn IN (cnt/2, cnt/2 + 1))
        rn IN (
            (cnt + 1) / 2,          -- integer division: floor of midpoint
            (cnt + 2) / 2           -- ceiling of midpoint (same as floor for odd cnt)
        )
        -- For odd cnt=5: (5+1)/2=3, (5+2)/2=3 → picks row 3 (once)
        -- For even cnt=4: (4+1)/2=2, (4+2)/2=3 → picks rows 2 and 3
)
SELECT
    department,
    AVG(salary)::NUMERIC(10,2) AS median_salary
FROM   median_rows
GROUP  BY department
ORDER  BY department;

-- Expected output:
-- department  | median_salary
-- ------------+--------------
-- Engineering |   105000.00
-- HR          |    70000.00
-- Marketing   |    82500.00
```

**How the row selection formula works:**

| dept        | cnt | (cnt+1)/2 | (cnt+2)/2 | rows selected | median calc |
|---|---|---|---|---|---|
| Engineering | 5   | 3         | 3         | row 3 only    | 105000 |
| Marketing   | 4   | 2         | 3         | rows 2 and 3  | (85000+80000)/2 = 82500 |
| HR          | 1   | 1         | 1         | row 1 only    | 70000 |

**Approach B — PERCENTILE_CONT (built-in, one line):**

```sql
SELECT
    department,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) AS median_salary
FROM   salaries
GROUP  BY department
ORDER  BY department;

-- Same result as above, much simpler.
```

`PERCENTILE_CONT(0.5)` computes the 50th percentile with linear interpolation — equivalent to the mathematical median for even-count groups (averages the two middle values). `PERCENTILE_DISC(0.5)` instead returns the nearest actual value without interpolation.

**When the manual approach is needed:** The interviewer may ban built-in percentile functions to test whether you can implement the logic from scratch. In production, always use `PERCENTILE_CONT`.

**Follow-up questions the interviewer will ask:**

1. **"What's the difference between PERCENTILE_CONT and PERCENTILE_DISC?"**
   `PERCENTILE_CONT` interpolates — for a 4-row group with values 75k, 80k, 85k, 90k it returns 82,500 (average of the two middle values). `PERCENTILE_DISC` returns the nearest actual data point — it would return 80,000 (the lower middle value). Use CONT for true median, DISC when you need an actual observed value.

2. **"How would you compute the 25th and 75th percentile (IQR) in the same query?"**
   ```sql
   SELECT
       department,
       PERCENTILE_CONT(0.25) WITHIN GROUP (ORDER BY salary) AS p25,
       PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY salary) AS median,
       PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY salary) AS p75
   FROM   salaries
   GROUP  BY department;
   ```

3. **"Why not just use AVG for salary analysis?"**
   Salaries are skewed by outliers (one very high earner raises the mean significantly). The median is robust to outliers and more representative of the "typical" employee salary.

**Common mistakes that get you rejected:**
- Using `(cnt/2)` for odd counts — integer division of 5/2 = 2, which picks row 2 not row 3.
- The formula `rn = cnt/2 OR rn = cnt/2 + 1` works for even counts but picks row 2 (wrong) for odd count 5 — use `(cnt+1)/2, (cnt+2)/2` which handles both cases.
- Forgetting `AVG()` in the final select — for odd counts you get one row so AVG is a no-op, but for even counts you need AVG to average the two middle values.
- Confusing `PERCENTILE_CONT` (ordered-set aggregate, uses `WITHIN GROUP`) with regular window functions — it cannot be used with `OVER`.

---

### Q28: Customers Who Bought Product A But Not Product B
**Company:** Amazon, Flipkart  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** OA/Phone

**Question:** From a `purchases` table `(user_id, product)`, find all users who bought product 'Laptop' but never bought product 'Mouse'. Show three approaches: EXCEPT, NOT EXISTS, and LEFT JOIN IS NULL. Explain which is fastest and the dangerous NULL edge case with NOT IN.

**What interviewer is testing:** Knowledge of set-difference operators, understanding of NULL semantics in NOT IN, and ability to explain query plan differences.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE purchases (
    purchase_id INT PRIMARY KEY,
    user_id     INT,
    product     VARCHAR(50)
);

INSERT INTO purchases VALUES
(1, 101, 'Laptop'),
(2, 101, 'Mouse'),    -- user 101 bought both → should NOT appear
(3, 102, 'Laptop'),   -- user 102 only bought Laptop → SHOULD appear
(4, 103, 'Mouse'),    -- user 103 only bought Mouse → should NOT appear
(5, 104, 'Laptop'),
(6, 104, 'Keyboard'), -- user 104 bought Laptop + Keyboard (not Mouse) → SHOULD appear
(7, 105, 'Laptop'),
(8, 105, 'Mouse');    -- user 105 bought both → should NOT appear
```

**Approach A — EXCEPT:**
```sql
SELECT user_id FROM purchases WHERE product = 'Laptop'
EXCEPT
SELECT user_id FROM purchases WHERE product = 'Mouse';

-- Result: 102, 104
```
EXCEPT automatically deduplicates both sides. Clean and readable.

**Approach B — NOT EXISTS (recommended):**
```sql
SELECT DISTINCT p.user_id
FROM   purchases p
WHERE  p.product = 'Laptop'
AND    NOT EXISTS (
    SELECT 1
    FROM   purchases p2
    WHERE  p2.user_id = p.user_id
    AND    p2.product = 'Mouse'
);

-- Result: 102, 104
```
NOT EXISTS short-circuits as soon as one matching row is found. With an index on `(user_id, product)`, this is typically the fastest approach.

**Approach C — LEFT JOIN IS NULL (anti-join):**
```sql
SELECT DISTINCT p.user_id
FROM   purchases p
LEFT JOIN purchases mouse_purchases
    ON  mouse_purchases.user_id = p.user_id
    AND mouse_purchases.product = 'Mouse'
WHERE  p.product = 'Laptop'
AND    mouse_purchases.user_id IS NULL;  -- NULL means no matching Mouse row

-- Result: 102, 104
```
The LEFT JOIN returns all Laptop buyers; NULLs in the right side mean no Mouse purchase found.

**The NOT IN NULL trap — why it's dangerous:**
```sql
-- This looks correct but is subtly broken:
SELECT DISTINCT user_id
FROM purchases
WHERE product = 'Laptop'
AND user_id NOT IN (
    SELECT user_id FROM purchases WHERE product = 'Mouse'
);
```

If any user_id in the Mouse subquery is NULL, the entire NOT IN returns no rows at all. This is because `x NOT IN (1, 2, NULL)` evaluates as `x != 1 AND x != 2 AND x != NULL`. Since `x != NULL` is always UNKNOWN (not TRUE), the entire expression is UNKNOWN, and the row is filtered out.

In this table `user_id` is NOT NULL, so NOT IN would work. But if `user_id` could be NULL, NOT IN silently returns zero results — a production disaster.

**Performance ranking (fastest to slowest):**

1. **NOT EXISTS** — with index on `(user_id, product)`, the inner check short-circuits immediately after finding one match. Query plan: index scan + nested loop.
2. **LEFT JOIN IS NULL** — hash join, processes all rows. Similar speed to NOT EXISTS, sometimes faster for large tables.
3. **EXCEPT** — materialises both result sets and computes the set difference with a sort or hash. Slightly more overhead due to deduplication of both sides.
4. **NOT IN** — can't use index efficiently for NULL-safe check; correlated execution.

**Follow-up questions the interviewer will ask:**

1. **"What if you want users who bought Laptop but not Mouse AND not Keyboard?"**
   ```sql
   SELECT user_id FROM purchases WHERE product = 'Laptop'
   EXCEPT
   (SELECT user_id FROM purchases WHERE product = 'Mouse'
    UNION
    SELECT user_id FROM purchases WHERE product = 'Keyboard');
   -- or using NOT EXISTS with OR:
   SELECT DISTINCT user_id FROM purchases p
   WHERE product = 'Laptop'
   AND NOT EXISTS (SELECT 1 FROM purchases p2
                   WHERE p2.user_id = p.user_id
                   AND p2.product IN ('Mouse','Keyboard'));
   ```

2. **"Does EXCEPT remove duplicates from the first set?"**
   Yes — `EXCEPT` is equivalent to `EXCEPT DISTINCT`. If user 102 bought Laptop twice (two rows), the result still returns 102 exactly once. Use `EXCEPT ALL` to preserve duplicates (rarely needed).

**Common mistakes that get you rejected:**
- Using NOT IN without checking whether the subquery can return NULLs — this is a well-known production bug pattern.
- Forgetting `DISTINCT` in the NOT EXISTS / LEFT JOIN approach — if a user bought Laptop 3 times, they appear 3 times.
- Confusing `EXCEPT` (set difference) with `MINUS` (Oracle syntax) — PostgreSQL uses `EXCEPT`.
- In the LEFT JOIN approach, putting `AND mouse_purchases.product = 'Mouse'` in the WHERE instead of in the ON clause — this turns the LEFT JOIN into an INNER JOIN, returning only rows with a Mouse match.

---

### Q29: Overlapping Interval Detection — Self-Join
**Company:** Google Calendar team, Hotstar  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** You have a `bookings` table `(id, room_id, start_time, end_time)`. Find all pairs of bookings for the same room that overlap in time. Two bookings overlap if `A.start_time < B.end_time AND A.end_time > B.start_time`. Report each pair only once.

**What interviewer is testing:** Self-join on a non-equality condition, the correct overlap predicate (not the common incorrect version), and deduplication of pairs.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE bookings (
    id         INT PRIMARY KEY,
    room_id    INT,
    start_time TIMESTAMPTZ,
    end_time   TIMESTAMPTZ,
    CHECK (end_time > start_time)
);

INSERT INTO bookings VALUES
(1, 101, '2024-01-01 09:00', '2024-01-01 10:00'),
(2, 101, '2024-01-01 09:30', '2024-01-01 11:00'),  -- overlaps with 1 (9:30 < 10:00)
(3, 101, '2024-01-01 10:00', '2024-01-01 11:00'),  -- touches 1 at 10:00 but does NOT overlap (boundary)
(4, 101, '2024-01-01 11:30', '2024-01-01 12:30'),  -- no overlap
(5, 102, '2024-01-01 09:00', '2024-01-01 10:00'),  -- different room, no overlap with room 101
(6, 102, '2024-01-01 09:15', '2024-01-01 09:45');  -- overlaps with 5 (same room 102)
```

**Core overlap query:**
```sql
SELECT
    a.id          AS booking_a,
    b.id          AS booking_b,
    a.room_id,
    a.start_time  AS a_start,
    a.end_time    AS a_end,
    b.start_time  AS b_start,
    b.end_time    AS b_end
FROM bookings a
JOIN bookings b
    ON  a.room_id = b.room_id          -- same room
    AND a.id < b.id                    -- each pair reported once (avoids (1,2) and (2,1))
    AND a.start_time < b.end_time      -- overlap condition left side
    AND a.end_time   > b.start_time    -- overlap condition right side
ORDER BY a.room_id, a.id, b.id;

-- Expected output:
-- booking_a | booking_b | room_id | a_start          | a_end            | b_start          | b_end
-- ----------+-----------+---------+------------------+------------------+------------------+------------------
--     1     |     2     |   101   | 2024-01-01 09:00 | 2024-01-01 10:00 | 2024-01-01 09:30 | 2024-01-01 11:00
--     5     |     6     |   102   | 2024-01-01 09:00 | 2024-01-01 10:00 | 2024-01-01 09:15 | 2024-01-01 09:45
```

**Why booking 3 does NOT overlap with booking 1:**
- Booking 1 ends at 10:00. Booking 3 starts at 10:00.
- Overlap condition: `A.end_time > B.start_time` → `10:00 > 10:00` → FALSE.
- Adjacent/touching intervals are NOT overlapping. If your domain requires treating touching intervals as overlapping, change `>` to `>=` and `<` to `<=`.

**The overlap predicate — memorise this:**

Two intervals [A.start, A.end) and [B.start, B.end) overlap if and only if:
```
A.start < B.end  AND  A.end > B.start
```

Equivalently: they do NOT overlap if `A.end <= B.start` (A finishes before B starts) OR `B.end <= A.start` (B finishes before A starts). Negate both: `NOT (A.end <= B.start OR B.end <= A.start)` = `A.end > B.start AND B.end > A.start` — same condition.

**Finding rooms with any conflict (just the room IDs):**
```sql
SELECT DISTINCT a.room_id
FROM bookings a
JOIN bookings b
    ON  a.room_id = b.room_id
    AND a.id < b.id
    AND a.start_time < b.end_time
    AND a.end_time   > b.start_time;
```

**Counting total conflicts per room:**
```sql
SELECT a.room_id, COUNT(*) AS conflict_count
FROM bookings a
JOIN bookings b
    ON  a.room_id = b.room_id
    AND a.id < b.id
    AND a.start_time < b.end_time
    AND a.end_time   > b.start_time
GROUP BY a.room_id;
```

**Follow-up questions the interviewer will ask:**

1. **"How would you prevent overlapping bookings at INSERT time?"**
   Use a PostgreSQL exclusion constraint with the `btree_gist` extension:
   ```sql
   CREATE EXTENSION IF NOT EXISTS btree_gist;
   ALTER TABLE bookings
   ADD CONSTRAINT no_overlap
   EXCLUDE USING GIST (
       room_id WITH =,
       tstzrange(start_time, end_time, '[)') WITH &&
   );
   -- && is the "overlaps" operator for range types; this constraint fires on INSERT/UPDATE.
   ```

2. **"What if end_time can be NULL (open-ended booking)?"**
   Replace `end_time` comparisons: treat NULL end_time as infinity. `COALESCE(a.end_time, 'infinity'::TIMESTAMPTZ)`. Or use PostgreSQL's `tstzrange` type natively which handles open ranges.

3. **"Can you use a window function instead of a self-join?"**
   For detecting conflicts you still need the self-join — window functions compute aggregates relative to the current row but cannot compare arbitrary pairs. However, LAG can check if consecutive bookings (by start time) overlap, which works only if bookings within a room don't span across each other. The self-join is the general solution.

**Common mistakes that get you rejected:**
- Using `a.start_time <= b.start_time AND a.end_time >= b.start_time` — this only detects one direction of overlap and misses cases where B starts before A.
- Forgetting `a.id < b.id` — returns every conflicting pair twice `(1,2)` and `(2,1)`.
- Using `a.id != b.id` instead of `a.id < b.id` — still double-counts.
- Treating touching intervals as overlapping or non-overlapping without clarifying the requirement.

---

### Q30: Longest Consecutive Active Days Per User — Island Grouping
**Company:** Duolingo, Hotstar, Swiggy  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** You have a `user_activity` table `(user_id, activity_date)`. Each row means a user was active on that date. Find the length of each user's longest streak of consecutive active days.

**What interviewer is testing:** The classic "islands" problem using the ROW_NUMBER minus date subtraction technique — the most important grouping-without-a-key pattern in SQL interviews.

**Ideal Answer:**

**Setup:**
```sql
CREATE TABLE user_activity (
    user_id       INT,
    activity_date DATE,
    PRIMARY KEY (user_id, activity_date)
);

INSERT INTO user_activity VALUES
-- User 1: active Jan 1-3, gap, Jan 6-9, gap, Jan 12 → streaks: 3, 4, 1
(1, '2024-01-01'), (1, '2024-01-02'), (1, '2024-01-03'),
(1, '2024-01-06'), (1, '2024-01-07'), (1, '2024-01-08'), (1, '2024-01-09'),
(1, '2024-01-12'),
-- User 2: active Jan 1, Jan 3-5 → streaks: 1, 3
(2, '2024-01-01'),
(2, '2024-01-03'), (2, '2024-01-04'), (2, '2024-01-05'),
-- User 3: active every day Jan 1-7 → streak: 7
(3, '2024-01-01'), (3, '2024-01-02'), (3, '2024-01-03'), (3, '2024-01-04'),
(3, '2024-01-05'), (3, '2024-01-06'), (3, '2024-01-07');
```

**The island technique — step by step:**

The key insight: within a consecutive sequence, the difference between `activity_date` and `ROW_NUMBER()` (ordered by date) is constant. When the sequence breaks, the constant changes — creating a new "island group".

**Step 1 — assign row numbers and compute the island key:**
```sql
WITH numbered AS (
    SELECT
        user_id,
        activity_date,
        ROW_NUMBER() OVER (
            PARTITION BY user_id
            ORDER BY activity_date
        ) AS rn
    FROM user_activity
),
islands AS (
    SELECT
        user_id,
        activity_date,
        rn,
        -- Subtracting an integer from a date gives a date in PostgreSQL
        activity_date - rn * INTERVAL '1 day' AS island_key
        -- All dates in the same consecutive run produce the same island_key
    FROM numbered
),
```

**Why this works — example for user 1:**
```
activity_date | rn | rn days | island_key (date - rn days)
--------------+----+---------+----------------------------
2024-01-01    |  1 | 1 day   | 2023-12-31   ← island A
2024-01-02    |  2 | 2 days  | 2023-12-31   ← island A (same key)
2024-01-03    |  3 | 3 days  | 2023-12-31   ← island A
                -- gap Jan 4-5 --
2024-01-06    |  4 | 4 days  | 2024-01-02   ← island B (new key)
2024-01-07    |  5 | 5 days  | 2024-01-02   ← island B
2024-01-08    |  6 | 6 days  | 2024-01-02   ← island B
2024-01-09    |  7 | 7 days  | 2024-01-02   ← island B
                -- gap Jan 10-11 --
2024-01-12    |  8 | 8 days  | 2024-01-04   ← island C (new key)
```

**Step 2 — count streak length per island:**
```sql
streak_lengths AS (
    SELECT
        user_id,
        island_key,
        MIN(activity_date) AS streak_start,
        MAX(activity_date) AS streak_end,
        COUNT(*)           AS streak_length
    FROM   islands
    GROUP  BY user_id, island_key
)
```

**Step 3 — find the maximum streak per user:**
```sql
SELECT
    user_id,
    MAX(streak_length) AS longest_streak,
    -- optionally show the start and end dates of the longest streak:
    (ARRAY_AGG(streak_start ORDER BY streak_length DESC))[1] AS best_streak_start,
    (ARRAY_AGG(streak_end   ORDER BY streak_length DESC))[1] AS best_streak_end
FROM   streak_lengths
GROUP  BY user_id
ORDER  BY user_id;

-- Expected output:
-- user_id | longest_streak | best_streak_start | best_streak_end
-- --------+----------------+-------------------+----------------
--    1    |       4        |    2024-01-06     |   2024-01-09
--    2    |       3        |    2024-01-03     |   2024-01-05
--    3    |       7        |    2024-01-01     |   2024-01-07
```

**PostgreSQL date arithmetic note:**
```sql
-- activity_date - rn * INTERVAL '1 day' works, but cleaner:
activity_date - MAKE_INTERVAL(days => rn) AS island_key
-- or cast rn to integer and subtract directly (date - integer = date in PostgreSQL):
activity_date - rn AS island_key   -- simplest form; rn is already an integer
```

**Full query (concise version):**
```sql
WITH islands AS (
    SELECT
        user_id,
        activity_date,
        activity_date - ROW_NUMBER() OVER (
            PARTITION BY user_id ORDER BY activity_date
        )::INT AS island_key
    FROM user_activity
)
SELECT user_id, MAX(cnt) AS longest_streak
FROM (
    SELECT user_id, island_key, COUNT(*) AS cnt
    FROM   islands
    GROUP  BY user_id, island_key
) streaks
GROUP BY user_id
ORDER BY user_id;
```

**Follow-up questions the interviewer will ask:**

1. **"Why does subtracting ROW_NUMBER from the date give a constant for consecutive dates?"**
   Consecutive dates differ by exactly 1 day. ROW_NUMBER also increases by exactly 1 per row. So `date - row_number` stays constant: Jan 1 - 1 = Dec 31, Jan 2 - 2 = Dec 31, Jan 3 - 3 = Dec 31. When a date is skipped (gap), the date jumps by 2 but row number only increments by 1, so the difference changes — new island.

2. **"How would you handle duplicate dates in the activity table?"**
   Add `DISTINCT` before computing ROW_NUMBER: `SELECT DISTINCT user_id, activity_date FROM user_activity`. Duplicate dates would increment ROW_NUMBER without advancing the date, breaking the island key. The PRIMARY KEY in the setup prevents this, but in practice always add DISTINCT as a safeguard.

3. **"Can you solve this without window functions (e.g., MySQL 5.7)?"**
   Yes, using a self-join to check whether the previous day exists:
   ```sql
   SELECT a.user_id, a.activity_date
   FROM   user_activity a
   WHERE  NOT EXISTS (
       SELECT 1 FROM user_activity b
       WHERE  b.user_id = a.user_id
       AND    b.activity_date = a.activity_date - INTERVAL '1 day'
   );
   -- Returns streak-start dates. Then measure streak length from each start date.
   ```
   This is significantly more complex to complete; the ROW_NUMBER approach is far superior.

**Common mistakes that get you rejected:**
- Using `activity_date - ROW_NUMBER()` with a DATE column and an INTEGER — in PostgreSQL this works directly (`DATE - INTEGER = DATE`), but in MySQL you need `DATE_SUB(activity_date, INTERVAL rn DAY)`. Know your dialect.
- Forgetting `PARTITION BY user_id` in ROW_NUMBER — row numbers span all users, completely breaking the island keys.
- Using `activity_date - LAG(activity_date)` to detect gaps and then trying to assign group IDs — this only flags gap starts; you still need a cumulative SUM of flags to assign group IDs (similar to the session identification pattern in Q24).
- Solving this with a recursive CTE instead of the island technique — recursive CTEs work but are O(n × streak_length) and will time out for large datasets.

---

### Q31: CUME_DIST and PERCENT_RANK — Salary Percentile Banding
**Company:** PayPal, Razorpay  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** You have an `employees` table. Use `CUME_DIST()` and `PERCENT_RANK()` to show each employee's salary percentile, and then bucket employees into "Top 10%", "10–25%", and "Bottom 75%" bands. Explain the difference between the two functions.

**What interviewer is testing:** Deep knowledge of the less-common window functions beyond `RANK`/`ROW_NUMBER`. Candidates who only know `RANK` fail here; knowing when to use `CUME_DIST` vs `PERCENT_RANK` separates strong analysts from average ones.

**Ideal Answer:**

```sql
-- Setup
CREATE TABLE employees (
    emp_id     INT PRIMARY KEY,
    name       TEXT,
    department TEXT,
    salary     NUMERIC(10,2)
);

INSERT INTO employees VALUES
(1,  'Alice',   'Engineering', 120000),
(2,  'Bob',     'Engineering',  95000),
(3,  'Carol',   'Engineering',  95000),
(4,  'Dave',    'Engineering',  80000),
(5,  'Eve',     'Engineering',  80000),
(6,  'Frank',   'Sales',        70000),
(7,  'Grace',   'Sales',        65000),
(8,  'Heidi',   'Sales',        60000),
(9,  'Ivan',    'Sales',        55000),
(10, 'Judy',    'Sales',        50000);
```

```sql
-- CUME_DIST vs PERCENT_RANK, with percentile band
SELECT
    name,
    salary,
    ROUND(CUME_DIST()    OVER (ORDER BY salary)::NUMERIC, 4) AS cume_dist,
    ROUND(PERCENT_RANK() OVER (ORDER BY salary)::NUMERIC, 4) AS pct_rank,
    CASE
        WHEN CUME_DIST() OVER (ORDER BY salary) >= 0.90 THEN 'Top 10%'
        WHEN CUME_DIST() OVER (ORDER BY salary) >= 0.75 THEN '10–25%'
        ELSE 'Bottom 75%'
    END AS salary_band
FROM employees
ORDER BY salary DESC;
```

```
-- Expected output (10 rows, ordered by salary DESC):
-- name   | salary | cume_dist | pct_rank | salary_band
-- -------|--------|-----------|----------|------------
-- Alice  | 120000 | 1.0000    | 1.0000   | Top 10%
-- Bob    |  95000 | 0.8000    | 0.6667   | 10–25%
-- Carol  |  95000 | 0.8000    | 0.6667   | 10–25%
-- Dave   |  80000 | 0.5000    | 0.3333   | Bottom 75%
-- Eve    |  80000 | 0.5000    | 0.3333   | Bottom 75%
-- Frank  |  70000 | 0.3000    | 0.2222   | Bottom 75%
-- Grace  |  65000 | 0.2000    | 0.1111   | Bottom 75%
-- Heidi  |  60000 | 0.1000    | 0.0000   | Bottom 75%
-- Ivan   |  55000 | 0.0000    | 0.0000   | Bottom 75%
-- Judy   |  50000 | 0.0000    | 0.0000   | Bottom 75%
```

**Key difference:**
- `CUME_DIST()` = (number of rows with value ≤ current row) / total rows. The minimum value is `1/N`, never 0.
- `PERCENT_RANK()` = (rank − 1) / (total rows − 1). The first row always gets 0.0.
- For ties, both functions assign the same value to all tied rows, but `CUME_DIST` uses the *last* position of the tie group while `PERCENT_RANK` uses the *first*.

```sql
-- Simplified band using a subquery/CTE to avoid repeating the window:
WITH ranked AS (
    SELECT
        name,
        salary,
        CUME_DIST() OVER (ORDER BY salary) AS cd
    FROM employees
)
SELECT
    name,
    salary,
    ROUND(cd::NUMERIC, 4) AS cume_dist,
    CASE
        WHEN cd >= 0.90 THEN 'Top 10%'
        WHEN cd >= 0.75 THEN '10–25%'
        ELSE 'Bottom 75%'
    END AS salary_band
FROM ranked
ORDER BY salary DESC;
```

**Follow-up questions the interviewer will ask:**
1. **"Why use `CUME_DIST` for bands instead of `PERCENT_RANK`?"** — `CUME_DIST` is more intuitive for banding: a value of 0.9 means "90% of the population earns ≤ this salary." `PERCENT_RANK` gives 0 to the lowest earner, which can be confusing when you want "bottom X%."
2. **"What happens with ties in `CUME_DIST`?"** — All tied rows get the value of the *highest* rank in the tie group. So if 3 people tie for ranks 4–6, all three get `cume_dist = 6/N`. This means the band boundaries may not cut at exactly 10%.
3. **"How would you do this per department instead of globally?"** — Add `PARTITION BY department` inside the `OVER` clause: `CUME_DIST() OVER (PARTITION BY department ORDER BY salary)`.

**Common mistakes that get you rejected:**
- Confusing `CUME_DIST` with `PERCENT_RANK` — not being able to explain the off-by-one at the bottom (PERCENT_RANK gives 0, CUME_DIST gives 1/N).
- Putting `CUME_DIST() OVER (ORDER BY salary DESC)` and then checking `>= 0.90` — this reverses the distribution; descending order means the highest salary gets 1/N, giving wrong bands.
- Repeating the full window expression in every `CASE` branch instead of computing it once in a CTE.
- Forgetting that band thresholds are approximate when there are ties at the boundary value.

---

### Q32: LAG for Stock Price Change — Daily Absolute and Percentage Change
**Company:** Zerodha, Groww, Goldman Sachs  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** You have a `stock_prices` table with daily closing prices per stock symbol. Compute the daily absolute price change and the percentage change for each stock. Handle the first trading day of each stock gracefully (no prior row).

**What interviewer is testing:** Practical use of `LAG()` with `PARTITION BY` on real financial data, NULL handling, and percentage calculation. Very common in fintech interviews.

**Ideal Answer:**

```sql
-- Setup
CREATE TABLE stock_prices (
    symbol      TEXT,
    price_date  DATE,
    close_price NUMERIC(10,2),
    PRIMARY KEY (symbol, price_date)
);

INSERT INTO stock_prices VALUES
('AAPL', '2024-01-02', 185.20),
('AAPL', '2024-01-03', 184.25),
('AAPL', '2024-01-04', 182.68),
('AAPL', '2024-01-05', 187.15),
('AAPL', '2024-01-08', 188.32),
('MSFT', '2024-01-02', 374.02),
('MSFT', '2024-01-03', 373.50),
('MSFT', '2024-01-04', 370.95),
('MSFT', '2024-01-05', 375.80);
```

```sql
-- Daily absolute change and percentage change
SELECT
    symbol,
    price_date,
    close_price,
    LAG(close_price, 1) OVER (
        PARTITION BY symbol
        ORDER BY price_date
    )                                                           AS prev_close,

    close_price
        - LAG(close_price, 1) OVER (
              PARTITION BY symbol ORDER BY price_date
          )                                                     AS abs_change,

    ROUND(
        (close_price
            - LAG(close_price, 1) OVER (
                  PARTITION BY symbol ORDER BY price_date
              )
        )
        / NULLIF(
              LAG(close_price, 1) OVER (
                  PARTITION BY symbol ORDER BY price_date
              ), 0
          ) * 100,
    2)                                                          AS pct_change
FROM stock_prices
ORDER BY symbol, price_date;
```

```
-- Expected output:
-- symbol | price_date | close_price | prev_close | abs_change | pct_change
-- -------|------------|-------------|------------|------------|----------
-- AAPL   | 2024-01-02 | 185.20      | NULL       | NULL       | NULL
-- AAPL   | 2024-01-03 | 184.25      | 185.20     | -0.95      | -0.51
-- AAPL   | 2024-01-04 | 182.68      | 184.25     | -1.57      | -0.85
-- AAPL   | 2024-01-05 | 187.15      | 182.68     |  4.47      |  2.45
-- AAPL   | 2024-01-08 | 188.32      | 187.15     |  1.17      |  0.63
-- MSFT   | 2024-01-02 | 374.02      | NULL       | NULL       | NULL
-- MSFT   | 2024-01-03 | 373.50      | 374.02     | -0.52      | -0.14
-- ...
```

**Cleaner CTE version (avoids repeating the window):**

```sql
WITH with_prev AS (
    SELECT
        symbol,
        price_date,
        close_price,
        LAG(close_price, 1) OVER (
            PARTITION BY symbol ORDER BY price_date
        ) AS prev_close
    FROM stock_prices
)
SELECT
    symbol,
    price_date,
    close_price,
    prev_close,
    close_price - prev_close                                   AS abs_change,
    ROUND(
        (close_price - prev_close) / NULLIF(prev_close, 0) * 100,
    2)                                                         AS pct_change
FROM with_prev
ORDER BY symbol, price_date;
```

**Handling the first row explicitly with COALESCE:**

```sql
-- If you want to show 0.00 instead of NULL for the first day:
SELECT
    symbol,
    price_date,
    close_price,
    COALESCE(close_price - LAG(close_price,1) OVER (
        PARTITION BY symbol ORDER BY price_date), 0)           AS abs_change
FROM stock_prices
ORDER BY symbol, price_date;
```

**Follow-up questions the interviewer will ask:**
1. **"What if trading days have gaps (weekends, holidays)? Does your query handle that?"** — Yes. `LAG` looks at the *previous physical row* per partition ordered by `price_date`, regardless of calendar gaps. It does not assume consecutive dates. If you need to compare against a specific calendar day (e.g., Monday's close vs. Friday's, not Thursday's), you'd need a `JOIN` on a trading calendar table.
2. **"Why `NULLIF(prev_close, 0)` instead of just dividing?"** — Prevents a division-by-zero error if a stock's previous close was 0 (e.g., a new listing priced at $0 in error). `NULLIF(x, 0)` returns NULL when x is 0, and NULL division produces NULL rather than an error.
3. **"How would you compute a 5-day rolling return instead of 1-day?"** — Change `LAG(close_price, 1)` to `LAG(close_price, 5)`. This gives you the price exactly 5 rows back per symbol.

**Common mistakes that get you rejected:**
- Missing `PARTITION BY symbol` — without it, `LAG` compares across different symbols, giving meaningless results.
- Using `close_price / prev_close - 1` without `NULLIF` — crashes or returns infinity when `prev_close` is NULL or 0.
- Trying to use `close_price - LAG(...) / LAG(...)` without parentheses — operator precedence gives wrong results (`-` before `/` in this case gives you `close_price - (change/prev)` rather than `(close_price - prev) / prev`).
- Not ordering by `price_date` inside the window — relying on insertion order is non-deterministic.

---

### Q33: STRING_AGG — Comma-Separated Product List Per Order
**Company:** Amazon, Flipkart  **Difficulty:** 🟢 Easy  **Frequency:** 🔥🔥🔥  **Round:** OA/Phone

**Question:** You have an `order_items` table. For each order, produce a single row with a comma-separated list of product names sorted alphabetically. Also show the `ARRAY_AGG` alternative and how to use `DISTINCT` within the aggregation.

**What interviewer is testing:** Knowledge of aggregate string functions, `ORDER BY` inside aggregates, and PostgreSQL array types. Extremely common in e-commerce SQL rounds.

**Ideal Answer:**

```sql
-- Setup
CREATE TABLE order_items (
    order_id     INT,
    product_name TEXT,
    quantity     INT
);

INSERT INTO order_items VALUES
(101, 'Keyboard',  1),
(101, 'Mouse',     2),
(101, 'Monitor',   1),
(102, 'Laptop',    1),
(102, 'Laptop',    1),   -- duplicate to demonstrate DISTINCT
(102, 'Charger',   1),
(103, 'Webcam',    1);
```

```sql
-- STRING_AGG: sorted comma-separated product list per order
SELECT
    order_id,
    STRING_AGG(product_name, ', ' ORDER BY product_name) AS products
FROM order_items
GROUP BY order_id
ORDER BY order_id;
```

```
-- Expected output:
-- order_id | products
-- ---------|-----------------------------------
-- 101      | Keyboard, Monitor, Mouse
-- 102      | Charger, Laptop, Laptop
-- 103      | Webcam
```

```sql
-- STRING_AGG with DISTINCT (removes duplicates before aggregating)
SELECT
    order_id,
    STRING_AGG(DISTINCT product_name, ', ' ORDER BY product_name) AS products_distinct
FROM order_items
GROUP BY order_id
ORDER BY order_id;
```

```
-- order_id | products_distinct
-- ---------|------------------
-- 101      | Keyboard, Monitor, Mouse
-- 102      | Charger, Laptop           -- Laptop appears only once
-- 103      | Webcam
```

```sql
-- ARRAY_AGG alternative: returns a PostgreSQL array instead of a string
SELECT
    order_id,
    ARRAY_AGG(product_name ORDER BY product_name) AS products_array,
    ARRAY_AGG(DISTINCT product_name ORDER BY product_name) AS products_array_distinct
FROM order_items
GROUP BY order_id
ORDER BY order_id;
```

```
-- order_id | products_array                   | products_array_distinct
-- ---------|----------------------------------|------------------------
-- 101      | {Keyboard,Monitor,Mouse}         | {Keyboard,Monitor,Mouse}
-- 102      | {Charger,Laptop,Laptop}          | {Charger,Laptop}
-- 103      | {Webcam}                         | {Webcam}
```

**When to choose each:**
- `STRING_AGG` — when the output needs to be a human-readable string (API response, report).
- `ARRAY_AGG` — when the output stays in PostgreSQL (feeds into `= ANY(array)` filters, or is processed by application code as a list).
- In MySQL/MariaDB, the equivalent is `GROUP_CONCAT(product_name ORDER BY product_name SEPARATOR ', ')`.

**Follow-up questions the interviewer will ask:**
1. **"What's the `ORDER BY` inside `STRING_AGG` for? Isn't there already an outer `ORDER BY`?"** — The `ORDER BY` inside the aggregate controls the *order elements are joined within the string for each group*. The outer `ORDER BY` controls the row order of the final result set. They are independent.
2. **"How do you handle NULL values in `STRING_AGG`?"** — `STRING_AGG` automatically ignores `NULL` values (all standard aggregate functions skip NULLs). If you want to include a placeholder, use `COALESCE(product_name, 'Unknown')` inside the call.
3. **"Is there a maximum length for the resulting string?"** — In PostgreSQL, `TEXT` has no hard limit (up to ~1 GB). In MySQL, `GROUP_CONCAT` is limited by `group_concat_max_len` (default 1024 bytes) — a common footgun in production.

**Common mistakes that get you rejected:**
- Writing `STRING_AGG(product_name ORDER BY product_name, ', ')` — the separator and the `ORDER BY` positions are swapped; correct syntax is `STRING_AGG(expr, separator ORDER BY ...)`.
- Assuming `GROUP_CONCAT` works in PostgreSQL — it is MySQL only. Using the wrong function in the wrong dialect immediately flags you.
- Not realizing `ARRAY_AGG` produces a PostgreSQL array type, not a string — trying to concatenate it with `||` on a VARCHAR column will fail with a type error.

---

### Q34: Reading an EXPLAIN ANALYZE Plan
**Company:** Any SDE-2 Onsite  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** You are given the following `EXPLAIN ANALYZE` output. Annotate every node: what does `cost=(X..Y)` mean, what does `rows=Z width=W` mean, what is `actual time`, and what does `loops` mean? When should you use `EXPLAIN` vs `EXPLAIN ANALYZE` vs `EXPLAIN (ANALYZE, BUFFERS)`?

**What interviewer is testing:** Whether you can actually diagnose a slow query in production — not just write SQL, but understand the query planner's decisions. This is a senior-level signal.

**Ideal Answer:**

**The sample plan to annotate:**

```
QUERY PLAN
-----------------------------------------------------------------------------
Hash Join  (cost=45.50..128.30 rows=500 width=48)
           (actual time=2.341..8.523 rows=483 loops=1)
  Hash Cond: (o.customer_id = c.id)
  ->  Seq Scan on orders o  (cost=0.00..62.50 rows=3250 width=28)
                             (actual time=0.021..1.432 rows=3250 loops=1)
  ->  Hash  (cost=32.00..32.00 rows=1080 width=20)
             (actual time=1.872..1.872 rows=1080 loops=1)
        Buckets: 2048  Batches: 1  Memory Usage: 72kB
        ->  Seq Scan on customers c  (cost=0.00..32.00 rows=1080 width=20)
                                      (actual time=0.015..0.821 rows=1080 loops=1)
              Filter: (status = 'active')
              Rows Removed by Filter: 420
Planning Time: 0.342 ms
Execution Time: 8.761 ms
```

**Node-by-node annotation:**

```
Hash Join  (cost=45.50..128.30 rows=500 width=48)
           (actual time=2.341..8.523 rows=483 loops=1)
```
- `cost=45.50..128.30` — Planner's estimate in arbitrary cost units. `45.50` = startup cost (time to return the first row); `128.30` = total cost (time to return all rows). These are *relative* numbers, not milliseconds.
- `rows=500` — Planner estimated 500 output rows.
- `width=48` — Estimated average row width in bytes.
- `actual time=2.341..8.523` — Real wall-clock milliseconds. `2.341` ms to first row; `8.523` ms total for this node.
- `rows=483` — Actual rows produced. Compare to estimated `rows=500`: close, planner was accurate.
- `loops=1` — This node executed once. If inside a nested loop, this would be > 1 and `actual time` is per-iteration.

```
  ->  Seq Scan on orders o  (cost=0.00..62.50 rows=3250 width=28)
                             (actual time=0.021..1.432 rows=3250 loops=1)
```
- `Seq Scan` — Full sequential scan; reads every page of the `orders` table in physical order. Chosen because either there is no usable index, or the planner decided reading all rows is cheaper than random I/O.
- Startup cost `0.00` — Sequential scans can return first row immediately.

```
        Buckets: 2048  Batches: 1  Memory Usage: 72kB
```
- `Batches: 1` — The entire hash table fits in `work_mem`. If `Batches > 1`, the hash spilled to disk — increase `work_mem` to fix this.

```
              Filter: (status = 'active')
              Rows Removed by Filter: 420
```
- `420` rows were read but discarded by the `WHERE status = 'active'` filter. If this number is large relative to rows returned, an index on `status` may help.

**EXPLAIN variants:**

| Command | Runs query? | Shows actual times? | Shows buffer I/O? | Use when |
|---|---|---|---|---|
| `EXPLAIN` | No | No | No | Safe for production; check plan without executing |
| `EXPLAIN ANALYZE` | Yes | Yes | No | Development; diagnose slow queries; query actually runs |
| `EXPLAIN (ANALYZE, BUFFERS)` | Yes | Yes | Yes | Diagnose I/O bottlenecks; see cache hits vs disk reads |
| `EXPLAIN (ANALYZE, BUFFERS, FORMAT JSON)` | Yes | Yes | Yes | Machine-readable; feed into `pev2` or `explain.dalibo.com` |

**Warning:** Never run `EXPLAIN ANALYZE` on a destructive statement (`UPDATE`, `DELETE`, `INSERT`) in production without wrapping it in `BEGIN; ... ROLLBACK;`.

```sql
-- Safe way to EXPLAIN ANALYZE a mutating statement:
BEGIN;
EXPLAIN ANALYZE UPDATE orders SET status = 'processed' WHERE id = 1;
ROLLBACK;
```

**Red flags to look for in a plan:**
- `Seq Scan` on a large table with high `Rows Removed by Filter` → needs an index.
- `Hash Join` with `Batches > 1` → increase `work_mem`.
- `rows=1` estimated but `rows=10000` actual → statistics are stale; run `ANALYZE tablename`.
- `Nested Loop` joining two large tables → planner got row count wrong; check `enable_nestloop`.

**Follow-up questions the interviewer will ask:**
1. **"The planner estimated 500 rows but you got 483. That's fine. What if the estimate was 5 vs actual 50,000?"** — That is a catastrophic mis-estimate. It usually means table statistics are stale (`ANALYZE` hasn't run), or there is a data skew the planner's histogram doesn't capture. Fix: run `ANALYZE`, increase `default_statistics_target` for that column, or use a partial index.
2. **"How do you read a plan with `loops > 1`?"** — `actual time` is the per-loop average. Multiply by `loops` to get total time for that node. `rows` is also per-loop; total rows = `rows × loops`.
3. **"What tools can you use to visualize an EXPLAIN output?"** — `pev2` (online), `explain.dalibo.com`, `pgAdmin`'s graphical explain, or the `auto_explain` extension which logs slow-query plans automatically.

**Common mistakes that get you rejected:**
- Treating `cost` values as milliseconds — they are unit-less planner cost units, not wall-clock time.
- Not knowing that `loops > 1` means `actual time` must be multiplied to get total — this is the single most common misread.
- Saying "Seq Scan is always bad" — for small tables or queries returning > ~5–10% of rows, a Seq Scan is often *faster* than an index scan due to sequential I/O vs. random I/O.
- Not knowing about `ROLLBACK` wrapping for `EXPLAIN ANALYZE` on mutating queries in production.

---

### Q35: CTE vs Subquery vs Temp Table — Optimizer Behaviour and When to Use Each
**Company:** Uber, Swiggy, Zomato  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Compare CTEs, subqueries (derived tables / inline views), and temporary tables. When does PostgreSQL materialize a CTE? What changed in PostgreSQL 12? When should you force materialization with the `MATERIALIZED` keyword? When should you use a temp table instead of a CTE?

**What interviewer is testing:** Whether you understand that SQL is declarative but the optimizer has real execution-model implications — a common source of production performance bugs. Senior engineers must know this.

**Ideal Answer:**

**1. Subquery / Derived Table (inline view)**

```sql
-- Derived table: the inner SELECT is inlined into the outer query plan
-- The optimizer can push predicates INTO it ("predicate pushdown")
SELECT d.department, d.avg_salary
FROM (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
) d
WHERE d.avg_salary > 80000;
-- Optimizer may rewrite this as:
--   SELECT department, AVG(salary) FROM employees
--   GROUP BY department HAVING AVG(salary) > 80000
-- because it can see inside the subquery.
```

**2. CTE (Common Table Expression)**

```sql
-- Before PostgreSQL 12: CTEs were ALWAYS materialized (optimization fence)
-- The optimizer could NOT push predicates into the CTE body.
-- PostgreSQL 12+: CTEs are NOT materialized by default if they are
-- referenced only once and have no side effects (e.g., no VOLATILE functions).

WITH dept_avg AS (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT department, avg_salary
FROM dept_avg
WHERE avg_salary > 80000;
-- PG 12+: optimizer inlines this CTE (same as derived table above).
-- PG 11 and below: CTE is materialized first, then filtered — full scan!
```

```sql
-- Force materialization even on PG 12+ (useful to prevent re-execution
-- of an expensive subquery that is referenced multiple times):
WITH dept_avg AS MATERIALIZED (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT d1.department, d1.avg_salary
FROM dept_avg d1
JOIN dept_avg d2 ON d1.avg_salary > d2.avg_salary * 0.9;
-- Without MATERIALIZED, PG 12 may execute the CTE body TWICE.
-- With MATERIALIZED, it executes once and caches the result.

-- Force NOT materialized (inline even a multi-reference CTE):
WITH dept_avg AS NOT MATERIALIZED (
    SELECT department, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY department
)
SELECT * FROM dept_avg WHERE avg_salary > 80000;
```

**3. Temporary Table**

```sql
-- Use a temp table when:
--   a) The CTE result is very large and referenced many times in multi-step ETL
--   b) You need to CREATE AN INDEX on the intermediate result
--   c) You are chaining multiple CTEs and the plan is getting complex

CREATE TEMP TABLE dept_avg_tmp AS
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;

-- Now you can index it:
CREATE INDEX ON dept_avg_tmp(department);

-- And reference it in multiple subsequent queries cheaply:
SELECT * FROM dept_avg_tmp WHERE avg_salary > 80000;
SELECT * FROM dept_avg_tmp ORDER BY avg_salary DESC LIMIT 5;

-- Temp tables are session-scoped and dropped automatically on session end.
-- Use ON COMMIT DROP inside a transaction if you want it dropped at commit.
```

**Decision framework:**

| Scenario | Use |
|---|---|
| Single-use subquery for readability | CTE (PG 12+ inlines it automatically) |
| Subquery referenced 2+ times, expensive to compute | `CTE AS MATERIALIZED` |
| Multi-step ETL, large intermediate result | Temp table + index |
| Need predicate pushdown into the subquery | Derived table or CTE (PG 12+ NOT MATERIALIZED) |
| Side-effectful CTE (`INSERT`/`UPDATE` in data-modifying CTE) | CTE is always materialized regardless of version |

**Follow-up questions the interviewer will ask:**
1. **"What is a data-modifying CTE?"** — PostgreSQL allows `INSERT`, `UPDATE`, `DELETE` inside a `WITH` clause. Example: `WITH deleted AS (DELETE FROM logs WHERE created_at < now() - interval '30 days' RETURNING *) INSERT INTO archive SELECT * FROM deleted;` — atomic move of old rows. These are always materialized.
2. **"You said temp tables are session-scoped. What about `ON COMMIT DELETE ROWS` vs `ON COMMIT DROP`?"** — `ON COMMIT DELETE ROWS` truncates the temp table at each `COMMIT` but keeps the structure (good for repeated use in loops). `ON COMMIT DROP` drops the whole table at `COMMIT`. Both require `CREATE TEMP TABLE ... ON COMMIT ...` syntax.
3. **"When would a CTE hurt performance that a derived table would not?"** — On PostgreSQL 11 and below, any CTE acts as an optimization fence — the outer `WHERE` predicate cannot be pushed inside. `WHERE cte.column = 'x'` filters after full materialization instead of during the scan.

**Common mistakes that get you rejected:**
- Assuming CTEs are always optimization fences in all PostgreSQL versions — this was true before PG 12 but is not the default anymore.
- Not knowing the `MATERIALIZED` / `NOT MATERIALIZED` keywords exist — these are the levers to control the behavior explicitly.
- Saying "CTEs are always faster because they cache results" — this is only true when you deliberately use `MATERIALIZED` for a multi-referenced expensive expression.
- Confusing a temp table with a CTE — temp tables survive across multiple SQL statements in the same session; CTEs are only visible within the single query they are defined in.

---

### Q36: Upsert — INSERT ON CONFLICT for Idempotent Writes
**Company:** PayPal, PhonePe, Razorpay  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** You have a `user_scores` table. Implement an upsert: insert a new score, but if the user already exists, update their score only if the new score is higher. Show `ON CONFLICT DO NOTHING`, `ON CONFLICT DO UPDATE`, the `EXCLUDED` pseudo-table, and a partial-index conflict target. Explain why idempotency matters in payment systems.

**What interviewer is testing:** Knowledge of PostgreSQL's upsert syntax (not a standard SQL feature), understanding of idempotency as a system design concept, and the `EXCLUDED` pseudo-table.

**Ideal Answer:**

```sql
-- Setup
CREATE TABLE user_scores (
    user_id    INT PRIMARY KEY,
    username   TEXT NOT NULL,
    score      INT  NOT NULL DEFAULT 0,
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO user_scores (user_id, username, score) VALUES
(1, 'alice', 500),
(2, 'bob',   300),
(3, 'carol', 750);
```

```sql
-- 1. ON CONFLICT DO NOTHING: silently ignore duplicate key violations
INSERT INTO user_scores (user_id, username, score)
VALUES (1, 'alice', 600)
ON CONFLICT (user_id) DO NOTHING;
-- Alice's score remains 500. No error, no update.
```

```sql
-- 2. ON CONFLICT DO UPDATE: use EXCLUDED to reference the incoming row
INSERT INTO user_scores (user_id, username, score)
VALUES (1, 'alice', 600)
ON CONFLICT (user_id)
DO UPDATE SET
    score      = EXCLUDED.score,
    updated_at = NOW();
-- Alice's score is now 600. EXCLUDED refers to the row that was *rejected*.
```

```sql
-- 3. Conditional update: only raise the score, never lower it
INSERT INTO user_scores (user_id, username, score)
VALUES (1, 'alice', 450)
ON CONFLICT (user_id)
DO UPDATE SET
    score      = GREATEST(user_scores.score, EXCLUDED.score),
    updated_at = NOW()
WHERE EXCLUDED.score > user_scores.score;
-- Alice's score stays at 600 because 450 < 600.
-- The WHERE clause on DO UPDATE makes this a no-op if condition is false.
```

```sql
-- 4. Partial unique index conflict target
-- Use case: a user can have at most one "active" subscription per product.
CREATE TABLE subscriptions (
    user_id    INT,
    product_id INT,
    status     TEXT,   -- 'active' | 'cancelled'
    expires_at DATE
);
CREATE UNIQUE INDEX uq_active_sub
    ON subscriptions (user_id, product_id)
    WHERE status = 'active';

INSERT INTO subscriptions (user_id, product_id, status, expires_at)
VALUES (1, 42, 'active', '2025-12-31')
ON CONFLICT ON CONSTRAINT uq_active_sub   -- or: ON CONFLICT (user_id, product_id) WHERE status = 'active'
DO UPDATE SET expires_at = EXCLUDED.expires_at;
-- Renews existing active subscription instead of inserting a duplicate.
```

**Idempotency in payment systems:**

```
Idempotency: applying the same operation multiple times produces the same result as applying it once.

Why it matters in payments:
- A payment request may be retried due to network timeout.
- Without idempotency: user is charged twice.
- With upsert + idempotency key:
```

```sql
CREATE TABLE payment_events (
    idempotency_key TEXT PRIMARY KEY,   -- UUID sent by client
    user_id         INT,
    amount          NUMERIC(10,2),
    status          TEXT,
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- Client retries the same payment with the same idempotency_key:
INSERT INTO payment_events (idempotency_key, user_id, amount, status)
VALUES ('uuid-abc-123', 42, 99.99, 'pending')
ON CONFLICT (idempotency_key) DO NOTHING;
-- Second (and subsequent) retries are silently ignored.
-- The payment is processed exactly once.
```

**Follow-up questions the interviewer will ask:**
1. **"What is the `EXCLUDED` table?"** — It is a pseudo-table representing the row that *failed* to insert (i.e., the incoming row that caused the conflict). It has the same columns as the target table. You reference it in the `DO UPDATE SET` clause to access the new values.
2. **"Can you upsert multiple rows at once?"** — Yes: `INSERT INTO ... VALUES (...), (...), ... ON CONFLICT ... DO UPDATE ...` applies the conflict resolution to each row individually.
3. **"What's the difference between `ON CONFLICT (column)` and `ON CONFLICT ON CONSTRAINT name`?"** — `ON CONFLICT (column)` infers the constraint from the column list; it must match a unique index exactly. `ON CONFLICT ON CONSTRAINT name` targets a named constraint directly and is needed for partial indexes.

**Common mistakes that get you rejected:**
- Writing `ON DUPLICATE KEY UPDATE` (MySQL syntax) in a PostgreSQL interview — immediately flags you as not knowing the dialect.
- Forgetting that `EXCLUDED` refers to the *incoming* row, not the existing row — confusing `EXCLUDED.score` (new) with `user_scores.score` (existing) gives wrong update logic.
- Not knowing that the `WHERE` clause on `DO UPDATE` makes the update conditional — without it, every conflict always triggers the update.
- Claiming `ON CONFLICT DO NOTHING` returns the existing row — it does not; it returns 0 rows affected. To get the existing row back, use `INSERT ... ON CONFLICT DO UPDATE SET col = EXCLUDED.col RETURNING *` (always returns the row).

---

### Q37: Window Frame ROWS vs RANGE — Behaviour with Duplicate ORDER BY Values
**Company:** Any SDE-2 Analytical Round  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** You have a `daily_sales` table with duplicate sale dates. Show how `SUM() OVER (ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` differs from `SUM() OVER (ORDER BY sale_date RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)` when the `ORDER BY` column has ties. Explain which one gives the correct running total.

**What interviewer is testing:** Very few candidates know this gotcha. It separates engineers who have debugged window function output in production from those who have only read about them.

**Ideal Answer:**

```sql
-- Setup: note deliberate duplicate sale_dates
CREATE TABLE daily_sales (
    sale_date DATE,
    amount    NUMERIC(10,2)
);

INSERT INTO daily_sales VALUES
('2024-01-01', 100.00),
('2024-01-02', 200.00),
('2024-01-02', 150.00),   -- duplicate date
('2024-01-03', 300.00),
('2024-01-04', 250.00);
```

```sql
-- ROWS: physical row position — each row's frame ends at its own physical row
SELECT
    sale_date,
    amount,
    SUM(amount) OVER (
        ORDER BY sale_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_rows
FROM daily_sales
ORDER BY sale_date;
```

```
-- Expected output (ROWS):
-- sale_date  | amount | running_total_rows
-- -----------|--------|-------------------
-- 2024-01-01 | 100.00 | 100.00
-- 2024-01-02 | 200.00 | 300.00   ← includes rows 1–2
-- 2024-01-02 | 150.00 | 450.00   ← includes rows 1–3 (different from the other Jan-02 row!)
-- 2024-01-03 | 300.00 | 750.00
-- 2024-01-04 | 250.00 | 1000.00
```

```sql
-- RANGE: logical range — all rows with the same ORDER BY value as current row
-- are included in the frame together. Both Jan-02 rows get the SAME running total.
SELECT
    sale_date,
    amount,
    SUM(amount) OVER (
        ORDER BY sale_date
        RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS running_total_range
FROM daily_sales
ORDER BY sale_date;
```

```
-- Expected output (RANGE):
-- sale_date  | amount | running_total_range
-- -----------|--------|--------------------
-- 2024-01-01 | 100.00 | 100.00
-- 2024-01-02 | 200.00 | 450.00   ← BOTH Jan-02 rows (200+150) are included for each Jan-02 row
-- 2024-01-02 | 150.00 | 450.00   ← same value — both rows see the full Jan-02 group
-- 2024-01-03 | 300.00 | 750.00
-- 2024-01-04 | 250.00 | 1000.00
```

**The key difference:**
- `ROWS BETWEEN ... CURRENT ROW` — the frame boundary is the *physical position* of the current row. Duplicate ORDER BY values produce different frame sizes and different running totals for each row in the tie group. Gives an *inconsistent* running total for duplicates.
- `RANGE BETWEEN ... CURRENT ROW` — the frame includes *all rows with the same ORDER BY value* as the current row. All tied rows see the same running total. This is the **default** when you write `ORDER BY` without an explicit frame clause.

**The default frame clause:**

```sql
-- These two are IDENTICAL — RANGE UNBOUNDED PRECEDING TO CURRENT ROW is the default:
SUM(amount) OVER (ORDER BY sale_date)
SUM(amount) OVER (ORDER BY sale_date RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)

-- To get ROWS behavior, you must say so explicitly:
SUM(amount) OVER (ORDER BY sale_date ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```

**When ROWS is correct:** When `ORDER BY` has a unique key (e.g., `ORDER BY id`), `ROWS` and `RANGE` produce identical results. Use `ROWS` explicitly for running totals over a unique key to get the most efficient plan (no peer-group lookup needed).

**Follow-up questions the interviewer will ask:**
1. **"What is the default frame when you write `ORDER BY` without `ROWS`/`RANGE`?"** — `RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`. This is why adding `ORDER BY` to a window without a frame clause changes results when there are ties.
2. **"What about `GROUPS` mode (PostgreSQL 11+)?"** — `GROUPS` is a third frame mode that counts *peer groups* (sets of tied rows) rather than physical rows or value ranges. `GROUPS BETWEEN 1 PRECEDING AND 1 FOLLOWING` includes the previous peer group, current peer group, and next peer group.
3. **"How would you avoid the tie problem for running totals entirely?"** — Ensure the `ORDER BY` in the window function uses a unique column (or composite of columns): `ORDER BY sale_date, ctid` (using the physical tuple ID) to break ties deterministically.

**Common mistakes that get you rejected:**
- Not knowing that `ORDER BY` in a window function implies a frame clause — thinking it just sorts rows without affecting which rows are included.
- Claiming `ROWS` is always better for running totals — it produces incorrect results when the ORDER BY column has duplicates.
- Confusing the window's `ORDER BY` (which defines the frame) with the query's final `ORDER BY` (which sorts the result set).
- Not knowing that `RANGE` is the default — writing `SUM(...) OVER (ORDER BY date)` and being surprised when duplicate dates produce equal running totals.

---

### Q38: Date and Time Operations — DATE_TRUNC, EXTRACT, AGE, Intervals, AT TIME ZONE
**Company:** Hotstar, BookMyShow, Zomato  **Difficulty:** 🟡 Medium  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** You have an `events` table with `TIMESTAMPTZ` columns. Demonstrate `DATE_TRUNC`, `EXTRACT`, `AGE()`, interval arithmetic, and `AT TIME ZONE`. Show the timezone conversion pitfall and how to find events in the last 30 days vs. the current calendar month.

**What interviewer is testing:** Practical datetime SQL — nearly every production system has timezone bugs. Candidates who confuse `TIMESTAMP` and `TIMESTAMPTZ` will write buggy code.

**Ideal Answer:**

```sql
-- Setup
CREATE TABLE events (
    event_id   SERIAL PRIMARY KEY,
    user_id    INT,
    event_name TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

INSERT INTO events (user_id, event_name, created_at) VALUES
(1, 'signup',    '2024-01-15 09:30:00+05:30'),
(1, 'purchase',  '2024-02-20 14:00:00+05:30'),
(2, 'signup',    '2024-03-01 11:00:00+05:30'),
(2, 'purchase',  '2024-03-10 16:45:00+05:30'),
(3, 'signup',    '2024-04-05 08:00:00+05:30');
```

```sql
-- DATE_TRUNC: truncate to a time unit boundary
SELECT
    created_at,
    DATE_TRUNC('day',   created_at)  AS day_start,    -- 2024-01-15 00:00:00+05:30
    DATE_TRUNC('month', created_at)  AS month_start,  -- 2024-01-01 00:00:00+05:30
    DATE_TRUNC('week',  created_at)  AS week_start,   -- Monday of that week
    DATE_TRUNC('hour',  created_at)  AS hour_start
FROM events
LIMIT 2;
```

```sql
-- EXTRACT / date_part: pull a numeric component from a timestamp
SELECT
    created_at,
    EXTRACT(YEAR   FROM created_at) AS yr,
    EXTRACT(MONTH  FROM created_at) AS mo,
    EXTRACT(DOW    FROM created_at) AS day_of_week,  -- 0=Sunday, 6=Saturday
    EXTRACT(EPOCH  FROM created_at) AS unix_epoch,   -- seconds since 1970-01-01 UTC
    EXTRACT(EPOCH  FROM (NOW() - created_at)) / 3600 AS hours_ago
FROM events;
```

```sql
-- AGE(): human-readable interval between two timestamps
SELECT
    user_id,
    created_at,
    AGE(NOW(), created_at) AS account_age,  -- e.g., "3 mons 12 days 04:15:00"
    AGE(created_at)        AS age_shorthand -- AGE(x) = AGE(NOW(), x)
FROM events
WHERE event_name = 'signup';
```

```sql
-- Interval arithmetic
SELECT
    created_at,
    created_at + INTERVAL '7 days'   AS plus_7_days,
    created_at - INTERVAL '1 month'  AS minus_1_month,
    created_at + INTERVAL '2 hours 30 minutes' AS plus_2h30m
FROM events
LIMIT 1;
```

```sql
-- Last 30 days (rolling window — always 30 days back from NOW):
SELECT * FROM events
WHERE created_at >= NOW() - INTERVAL '30 days';

-- Current calendar month (Jan 1 00:00 to Jan 31 23:59:59):
SELECT * FROM events
WHERE created_at >= DATE_TRUNC('month', NOW())
  AND created_at <  DATE_TRUNC('month', NOW()) + INTERVAL '1 month';
-- Note: use < start_of_next_month rather than <= end_of_current_month
-- to avoid microsecond boundary bugs with TIMESTAMPTZ.
```

```sql
-- AT TIME ZONE: convert between timezones
-- TIMESTAMPTZ stored as UTC internally; AT TIME ZONE renders it in a specific zone.
SELECT
    created_at                                    AS utc_stored,
    created_at AT TIME ZONE 'Asia/Kolkata'        AS ist_display,
    created_at AT TIME ZONE 'America/New_York'    AS ny_display
FROM events
LIMIT 1;

-- THE PITFALL:
-- TIMESTAMP (without TZ) + AT TIME ZONE INTERPRETS the timestamp as being in that zone.
-- TIMESTAMPTZ (with TZ)  + AT TIME ZONE CONVERTS the stored UTC to that zone.
-- These are opposite operations! Always use TIMESTAMPTZ for application data.

-- Dangerous pitfall example:
SELECT TIMESTAMP '2024-01-15 09:30:00' AT TIME ZONE 'UTC';
-- Returns: 2024-01-15 09:30:00+00  (interprets the TIMESTAMP as UTC → no conversion)
SELECT TIMESTAMPTZ '2024-01-15 09:30:00+00' AT TIME ZONE 'Asia/Kolkata';
-- Returns: 2024-01-15 15:00:00     (converts UTC to IST = UTC+5:30)
```

**Follow-up questions the interviewer will ask:**
1. **"What's the difference between `TIMESTAMP` and `TIMESTAMPTZ`?"** — `TIMESTAMP` stores the value as-is with no timezone context; it is "naive." `TIMESTAMPTZ` stores the value internally as UTC and records the offset; it is "aware." Always use `TIMESTAMPTZ` for user-facing event data to avoid bugs when the server timezone changes.
2. **"Why use `< DATE_TRUNC('month', NOW()) + INTERVAL '1 month'` instead of `<= last_day_of_month`?"** — `TIMESTAMPTZ` has microsecond precision. If you use `<= '2024-01-31 23:59:59'` you miss events from `23:59:59.000001` to `23:59:59.999999`. The half-open interval `[start, start_of_next_month)` is always correct.
3. **"How do you group events by week, starting Monday?"** — `DATE_TRUNC('week', created_at)` always truncates to Monday (ISO week). If you need Sunday-based weeks, use `DATE_TRUNC('week', created_at + INTERVAL '1 day') - INTERVAL '1 day'`.

**Common mistakes that get you rejected:**
- Using `TIMESTAMP` instead of `TIMESTAMPTZ` for user event timestamps — will produce wrong results when the server timezone or the application's session timezone changes.
- Comparing `created_at::DATE = CURRENT_DATE` instead of using a range — casting to `DATE` strips the timezone offset, which may shift the date; also prevents index use.
- Using `DATEDIFF` (MySQL) in a PostgreSQL interview — PostgreSQL uses subtraction: `date2 - date1` returns an `INTEGER` number of days for `DATE` types, or an `INTERVAL` for `TIMESTAMP` types.
- Forgetting that `EXTRACT(DOW ...)` returns 0 for Sunday (PostgreSQL convention), while `EXTRACT(ISODOW ...)` returns 7 for Sunday (ISO 8601).

---

### Q39: JSONB Columns — Operators, Containment, Array Unnesting, GIN Index
**Company:** Razorpay, Stripe-India, Startups  **Difficulty:** 🔴 Hard  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** You have a `products` table with a `jsonb` metadata column. Demonstrate the `->`, `->>`, `#>`, `#>>` operators, the `@>` containment operator, `jsonb_array_elements` to unnest an embedded array, and a GIN index. Explain the performance difference between `json` and `jsonb`.

**What interviewer is testing:** Whether you can work with semi-structured data in PostgreSQL without reaching for a NoSQL database. Companies storing flexible product metadata, payment event payloads, or feature flags in JSONB columns ask this.

**Ideal Answer:**

```sql
-- Setup
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    name       TEXT,
    metadata   JSONB
);

INSERT INTO products VALUES
(1, 'Laptop Pro',  '{"brand":"Apple","specs":{"ram":16,"storage":512},"tags":["laptop","premium","m2"],"price":129999}'),
(2, 'Budget Phone','{"brand":"Xiaomi","specs":{"ram":6,"storage":128},"tags":["phone","budget"],"price":12999}'),
(3, 'Wireless Ear','{"brand":"Sony","specs":{"battery":30,"anc":true},"tags":["audio","premium"],"price":19999}'),
(4, 'Gaming Mouse','{"brand":"Logitech","specs":{"dpi":25600},"tags":["gaming","peripheral"],"price":4999}');
```

```sql
-- -> operator: returns JSON object/array (preserves JSON type)
-- ->> operator: returns TEXT (extracts as a plain string)
SELECT
    product_id,
    metadata -> 'brand'                       AS brand_json,     -- "Apple"  (JSON string with quotes)
    metadata ->> 'brand'                      AS brand_text,     -- Apple    (plain text, no quotes)
    metadata -> 'specs' -> 'ram'              AS ram_json,       -- 16       (JSON number)
    (metadata -> 'specs' ->> 'ram')::INT      AS ram_int,        -- 16       (cast to integer for arithmetic)
    metadata ->> 'price'                      AS price_text      -- 129999   (as text)
FROM products;
```

```sql
-- #> operator: path navigation using an array of keys (returns JSON)
-- #>> operator: path navigation (returns TEXT)
SELECT
    product_id,
    metadata #>  '{specs,ram}'                AS ram_json,   -- path array navigation
    metadata #>> '{specs,ram}'                AS ram_text,   -- same but returns TEXT
    metadata #>> '{tags,0}'                   AS first_tag   -- first element of tags array
FROM products;
```

```sql
-- @> containment operator: does the left jsonb contain the right jsonb?
-- "Contains" means all key-value pairs on the right exist in the left.
SELECT product_id, name
FROM products
WHERE metadata @> '{"brand":"Apple"}';
-- Returns: Laptop Pro

SELECT product_id, name
FROM products
WHERE metadata @> '{"specs":{"ram":16}}';
-- Returns: Laptop Pro (nested containment works)

SELECT product_id, name
FROM products
WHERE metadata @> '{"tags":["premium"]}';
-- Returns: Laptop Pro AND Wireless Ear (array containment: 'premium' in tags)
```

```sql
-- jsonb_array_elements: unnest a JSON array into rows
-- Use case: find all products that have a specific tag
SELECT
    p.product_id,
    p.name,
    tag.value ->> 0                           AS tag_name   -- each element is a JSON string
FROM products p,
LATERAL jsonb_array_elements(p.metadata -> 'tags') AS tag(value)
ORDER BY p.product_id, tag_name;
```

```
-- Expected output (each tag becomes its own row):
-- product_id | name         | tag_name
-- -----------|--------------|----------
-- 1          | Laptop Pro   | laptop
-- 1          | Laptop Pro   | m2
-- 1          | Laptop Pro   | premium
-- 2          | Budget Phone | budget
-- 2          | Budget Phone | phone
-- 3          | Wireless Ear | audio
-- 3          | Wireless Ear | premium
-- 4          | Gaming Mouse | gaming
-- 4          | Gaming Mouse | peripheral
```

```sql
-- Correct form for unnesting a JSON string array:
-- jsonb_array_elements returns each element as a jsonb value.
-- For a string array, each element is '"laptop"' (with quotes) — use ->> or ::text to strip them.
SELECT
    p.product_id,
    p.name,
    tag::TEXT                                 AS tag_name_raw,   -- "laptop" (with quotes)
    tag #>> '{}'                              AS tag_name_clean  -- laptop   (strips quotes)
FROM products p,
LATERAL jsonb_array_elements(p.metadata -> 'tags') AS tag
ORDER BY p.product_id;
```

```sql
-- GIN index: accelerates @>, ?, ?|, ?& operators on JSONB
CREATE INDEX idx_products_metadata ON products USING GIN (metadata);

-- This index now makes the @> containment query above use an index scan:
EXPLAIN SELECT * FROM products WHERE metadata @> '{"brand":"Apple"}';
-- With GIN index: Bitmap Index Scan on idx_products_metadata (much faster on large tables)

-- GIN with jsonb_path_ops (smaller index, only supports @> operator):
CREATE INDEX idx_products_metadata_path ON products USING GIN (metadata jsonb_path_ops);

-- Index on a specific key (B-tree, for range queries on a known key):
CREATE INDEX idx_products_price ON products ((metadata ->> 'price'));
-- Supports: WHERE (metadata ->> 'price')::INT > 10000
```

**json vs jsonb performance difference:**

| Feature | `json` | `jsonb` |
|---|---|---|
| Storage | Stored as raw text, exact copy | Stored as parsed binary |
| Write speed | Faster (no parsing) | Slower (parses and stores binary) |
| Read speed | Slower (must re-parse every read) | Faster (no re-parsing) |
| Indexing | Cannot use GIN | Supports GIN, B-tree on extracted keys |
| Key order | Preserved | Not preserved (object keys sorted) |
| Duplicate keys | Kept (last-write wins on read) | Deduplicated (last value kept) |
| Use case | Audit logs needing exact text | Application data needing querying |

**Recommendation:** Always use `jsonb` unless you specifically need to preserve key insertion order or have a write-heavy log ingestion pipeline where parsing overhead matters.

**Follow-up questions the interviewer will ask:**
1. **"When would you NOT use JSONB and use a proper normalized column instead?"** — When you query the same field in `WHERE` clauses frequently and need range queries (e.g., `price > 10000`), a proper `NUMERIC` column with a B-tree index will always outperform even an indexed JSONB extraction. JSONB is best for fields that vary per row (product specs differ by category) or for optional attributes with many nulls.
2. **"What does the `?` operator do?"** — `metadata ? 'brand'` returns true if the key `'brand'` exists at the top level of the JSON object. `?|` checks if any of multiple keys exist; `?&` checks if all of them exist.
3. **"What is `jsonb_set` used for?"** — `jsonb_set(target, path, new_value)` returns a new JSONB value with the specified path updated. Example: `UPDATE products SET metadata = jsonb_set(metadata, '{price}', '9999') WHERE product_id = 4;` — updates just the price field without rewriting the entire JSON blob.

**Common mistakes that get you rejected:**
- Confusing `->` (returns JSON) with `->>` (returns text) — using `->` in a `WHERE` clause comparison with a plain string like `WHERE metadata -> 'brand' = 'Apple'` fails because `-> 'brand'` returns `"Apple"` (a JSON string value, not a SQL text value). You need `->>` or `@>`.
- Using `json` type instead of `jsonb` — cannot create GIN indexes on `json`, and repeated reads are slower.
- Not knowing `@>` is the containment operator — writing subquery-based JSON searches or `LIKE` on `metadata::text`, which is both slow and brittle.
- Forgetting that `jsonb_array_elements` returns each element as a `jsonb` value — for a string array element `"laptop"`, you need `element #>> '{}'` or `element::text` (which includes quotes) to get a plain string.

---

### Q40: LATERAL JOIN — Replace a Correlated Subquery for Top-N Per Group
**Company:** Google, Amazon, Flipkart (senior rounds)  **Difficulty:** ⚫ Expert  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** You have `categories` and `products` tables. Find the top 3 most expensive products in each category. Show the correlated subquery approach first, explain why it is slow, then show the `LATERAL JOIN` approach. Also demonstrate calling a set-returning function with `LATERAL`. Explain what makes `LATERAL` unique.

**What interviewer is testing:** Whether you know advanced join types beyond `INNER`/`LEFT`/`CROSS`. `LATERAL` is the clean, performant solution for top-N-per-group and replaces correlated subqueries. Knowing this separates senior engineers from mid-level ones.

**Ideal Answer:**

```sql
-- Setup
CREATE TABLE categories (
    category_id   INT PRIMARY KEY,
    category_name TEXT
);

CREATE TABLE products (
    product_id    INT PRIMARY KEY,
    category_id   INT REFERENCES categories(category_id),
    product_name  TEXT,
    price         NUMERIC(10,2)
);

INSERT INTO categories VALUES
(1, 'Electronics'),
(2, 'Clothing'),
(3, 'Books');

INSERT INTO products VALUES
(1,  1, 'Laptop',         89999),
(2,  1, 'Smartphone',     59999),
(3,  1, 'Tablet',         39999),
(4,  1, 'Smartwatch',     24999),
(5,  1, 'Earbuds',        14999),
(6,  2, 'Winter Jacket',  8999),
(7,  2, 'Running Shoes',  6499),
(8,  2, 'Denim Jeans',    2999),
(9,  2, 'T-Shirt',         999),
(10, 3, 'System Design',  2499),
(11, 3, 'Clean Code',     1999),
(12, 3, 'DDIA',           2799),
(13, 3, 'SQL Cookbook',   1499);
```

```sql
-- APPROACH 1: Correlated subquery (slow — executes once per product row)
SELECT p.category_id, p.product_name, p.price
FROM products p
WHERE (
    SELECT COUNT(*)
    FROM products p2
    WHERE p2.category_id = p.category_id
      AND p2.price > p.price        -- count of products more expensive than current
) < 3                               -- include only if fewer than 3 are more expensive
ORDER BY p.category_id, p.price DESC;
```

```
-- Why this is slow:
-- For every row in 'products', the database re-executes the inner SELECT.
-- With 1M products across 1000 categories, that is 1M subquery executions.
-- Even with an index on (category_id, price), this is O(N × log M) where N = total products.
```

```sql
-- APPROACH 2: LATERAL JOIN (fast — one scan per category, not per product)
SELECT c.category_name, top3.product_name, top3.price
FROM categories c
CROSS JOIN LATERAL (
    SELECT product_name, price
    FROM products p
    WHERE p.category_id = c.category_id
    ORDER BY price DESC
    LIMIT 3
) AS top3
ORDER BY c.category_name, top3.price DESC;
```

```
-- Expected output:
-- category_name | product_name   | price
-- --------------|----------------|-------
-- Books         | DDIA           | 2799
-- Books         | System Design  | 2499
-- Books         | Clean Code     | 1999
-- Clothing      | Winter Jacket  | 8999
-- Clothing      | Running Shoes  | 6499
-- Clothing      | Denim Jeans    | 2999
-- Electronics   | Laptop         | 89999
-- Electronics   | Smartphone     | 59999
-- Electronics   | Tablet         | 39999
```

**What makes LATERAL special:**

```
Normal subquery / derived table:
  Each subquery in FROM is evaluated independently.
  It cannot reference columns from other tables in the same FROM clause.
  FROM orders o, (SELECT * FROM items WHERE items.order_id = o.id) i  -- ERROR without LATERAL

LATERAL:
  The subquery can reference columns from preceding FROM items.
  Think of it as a "foreach loop":
    for each row in categories c:
        execute the LATERAL subquery with c.category_id substituted in.
  This is exactly like a correlated subquery, but in the FROM clause,
  allowing LIMIT, ORDER BY, aggregation, and function calls inside it.
```

```sql
-- LEFT JOIN LATERAL: include categories with fewer than 3 products (or zero products)
-- CROSS JOIN LATERAL drops categories with no matching products (like INNER JOIN).
SELECT c.category_name, top3.product_name, top3.price
FROM categories c
LEFT JOIN LATERAL (
    SELECT product_name, price
    FROM products p
    WHERE p.category_id = c.category_id
    ORDER BY price DESC
    LIMIT 3
) AS top3 ON TRUE           -- ON TRUE is required syntax for LEFT JOIN LATERAL
ORDER BY c.category_name, top3.price DESC NULLS LAST;
```

```sql
-- LATERAL with a set-returning function:
-- generate_series is a set-returning function; LATERAL allows it to reference outer columns.
SELECT
    c.category_name,
    gs.rank_slot
FROM categories c
CROSS JOIN LATERAL generate_series(1, 3) AS gs(rank_slot)
ORDER BY c.category_name, gs.rank_slot;
-- Produces one row per (category, rank_slot) pair — useful for building slot frames.

-- Real-world use: LATERAL with a function that parses each row's JSONB:
SELECT p.product_id, tag_elem
FROM products_with_tags p
CROSS JOIN LATERAL jsonb_array_elements_text(p.metadata -> 'tags') AS tag_elem
-- This is the idiomatic way to unnest arrays — equivalent to the LATERAL form we saw in Q39.
```

**Performance comparison with an index:**

```sql
-- Create a composite index to make LATERAL fast:
CREATE INDEX idx_products_cat_price ON products (category_id, price DESC);

-- The LATERAL subquery now uses an Index Scan on idx_products_cat_price,
-- reading exactly 3 rows per category. Total I/O = 3 × num_categories.

-- The correlated subquery approach cannot use this index as efficiently
-- because it needs a COUNT for every row, not just top-N retrieval.
```

**Follow-up questions the interviewer will ask:**
1. **"What is the difference between `CROSS JOIN LATERAL` and `LEFT JOIN LATERAL`?"** — `CROSS JOIN LATERAL` (equivalent to `INNER JOIN LATERAL`) drops the outer row if the lateral subquery returns zero rows. `LEFT JOIN LATERAL ... ON TRUE` keeps the outer row with NULLs for the lateral columns — used when some categories might have no products.
2. **"Can you use LATERAL with an UPDATE or DELETE?"** — Yes. PostgreSQL supports `UPDATE ... FROM LATERAL (...)` and `DELETE ... USING LATERAL (...)`. This is useful for batch operations where you need per-row computed values.
3. **"Is LATERAL standard SQL?"** — Yes, `LATERAL` was introduced in SQL:1999 and is supported in PostgreSQL, Oracle (12c+), and SQL Server (using `APPLY` keyword: `CROSS APPLY` / `OUTER APPLY`). MySQL added `LATERAL` in version 8.0.14.

**Common mistakes that get you rejected:**
- Not knowing that `LATERAL` exists and trying to solve top-N-per-group with a window function + CTE — the window approach works but is wordier and less flexible for variable N.
- Writing `LEFT JOIN LATERAL (...) AS x` without `ON TRUE` — this is a syntax error; `LEFT JOIN LATERAL` always requires `ON TRUE` (or a condition) since LATERAL is not a named table with a join key.
- Claiming `CROSS JOIN LATERAL` is the same as `CROSS JOIN` — a plain `CROSS JOIN` produces a Cartesian product of two independent sets; `CROSS JOIN LATERAL` executes the right side once per left row with the left row's columns in scope.
- Not creating the composite index `(category_id, price DESC)` when asked about performance — without it, the LATERAL subquery degenerates to a Seq Scan per category, and the correlated subquery might actually perform similarly.

---

### Q41: Explain the ACID Properties and Their Enforcement Mechanisms
**Company:** Any FAANG, Razorpay, PhonePe  **Difficulty:** 🟡  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** Explain each of the ACID properties. For each, describe the mechanism PostgreSQL uses to enforce it and give a failure scenario showing what breaks without it.

**What interviewer is testing:** Whether you understand transactions at the engine level, not just as a buzzword. Senior candidates can name the *mechanism* (WAL, MVCC, fsync), not just the definition.

**Ideal Answer:**

**A — Atomicity:** A transaction is all-or-nothing. Either every statement commits, or none do.
- *Mechanism:* PostgreSQL writes changes to the Write-Ahead Log (WAL). If a transaction aborts or the server crashes mid-transaction, recovery uses the WAL (and MVCC tuple visibility — uncommitted row versions are simply never made visible) to roll back. Each tuple carries an `xmin`; if that transaction never committed, the row is invisible — effectively undone.
- *Failure without it:* In a bank transfer, money is debited from account A but the credit to B fails. Without atomicity, $100 vanishes.

**C — Consistency:** A transaction moves the DB from one valid state to another, respecting all constraints.
- *Mechanism:* Constraint enforcement — `CHECK`, `NOT NULL`, `UNIQUE`, `FOREIGN KEY`, and triggers are validated at commit (or per-statement). Violations abort the transaction.
- *Failure without it:* A transfer leaves `balance = -50` even though a `CHECK (balance >= 0)` exists, corrupting business invariants.

**I — Isolation:** Concurrent transactions don't interfere; each sees a consistent snapshot.
- *Mechanism:* MVCC (Multi-Version Concurrency Control) plus row/predicate locks. Readers see a snapshot; writers create new tuple versions instead of overwriting.
- *Failure without it:* Two simultaneous withdrawals both read balance = $100, both subtract $100, and the account ends at $0 instead of -$100 being rejected — a lost update.

**D — Durability:** Once committed, data survives crashes/power loss.
- *Mechanism:* WAL is flushed to disk via `fsync` *before* commit is acknowledged (WAL redo on recovery replays committed-but-not-yet-checkpointed changes). Controlled by `synchronous_commit` and `wal_sync_method`.
- *Failure without it:* Commit returns success, power dies, the change was only in RAM — the customer's transfer is silently lost.

**Bank transfer showing all four matter:**

```sql
BEGIN;  -- Atomicity: both statements succeed or neither does

UPDATE accounts SET balance = balance - 100
WHERE id = 'A' AND balance >= 100;  -- Consistency: CHECK(balance>=0) won't be violated

UPDATE accounts SET balance = balance + 100
WHERE id = 'B';

-- Isolation: a concurrent transfer on A sees a consistent snapshot via MVCC;
--            row lock on A prevents a lost update.

COMMIT;  -- Durability: WAL fsync'd before this returns OK.
```

If the second `UPDATE` fails (e.g., B doesn't exist), the whole transaction rolls back (A) — A is never debited; the `CHECK` guards negative balances (C); MVCC + the row lock serialize concurrent transfers (I); and once `COMMIT` returns, a crash can't lose it (D).

**Follow-up questions the interviewer will ask:**

1. *"How does WAL provide BOTH atomicity and durability?"* — Durability: WAL is flushed before commit, so committed changes can be redone after a crash (redo log). Atomicity: on recovery, changes from transactions that never committed are not made visible / are rolled back (undo behavior comes from MVCC visibility + WAL). One log, two guarantees.

2. *"Can you weaken durability for performance?"* — Yes: `SET synchronous_commit = off` lets commits return before WAL is fsync'd. You risk losing the last few hundred ms of transactions on a crash, but you keep atomicity/consistency. Useful for non-critical bulk loads.

3. *"Does Consistency in ACID mean the same as Consistency in CAP?"* — No. ACID-C is about constraint validity within a single node. CAP-C is about all nodes seeing the same latest write in a distributed system. Mixing them up is a classic red flag.

**Common mistakes that get you rejected:**
- Defining the letters but not naming the mechanism (WAL/MVCC/fsync). Anyone can recite definitions.
- Confusing ACID Consistency with CAP Consistency.
- Saying "isolation means transactions run one at a time" — they run concurrently; isolation is about *appearing* serial.

---

### Q42: Explain the Four SQL Isolation Levels and the Anomalies They Prevent
**Company:** Amazon, Google, Flipkart  **Difficulty:** 🔴  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** Walk through READ UNCOMMITTED, READ COMMITTED, REPEATABLE READ, and SERIALIZABLE. For each, show which concurrency anomalies it allows or prevents using a two-transaction timeline.

**What interviewer is testing:** Deep concurrency knowledge. This separates people who memorized the table from people who can reason about *why* an anomaly happens.

**Ideal Answer:**

**The four anomalies:**
- **Dirty Read:** Reading another transaction's *uncommitted* changes.
- **Non-Repeatable Read:** Re-reading the *same row* returns a different value (another txn committed an UPDATE in between).
- **Phantom Read:** Re-running the *same query* returns a different *set of rows* (another txn committed an INSERT/DELETE matching the predicate).
- **Serialization Anomaly:** The result of committing a group of concurrent transactions is inconsistent with any serial order — even though no individual row anomaly occurred (e.g., write skew).

**READ UNCOMMITTED** — allows dirty reads (in the standard).

```
Transaction A                          Transaction B
-------------                          -------------
BEGIN;
UPDATE accounts SET bal=0 WHERE id=1;
                                       BEGIN;
                                       SELECT bal FROM accounts WHERE id=1;
                                       -- reads 0  (DIRTY READ!)
ROLLBACK;  -- bal was never really 0
```
B acted on data that never existed. **PostgreSQL does not implement READ UNCOMMITTED — it silently treats it as READ COMMITTED**, so dirty reads never happen in PostgreSQL.

**READ COMMITTED** (PostgreSQL default) — prevents dirty reads, allows non-repeatable + phantom reads. Each *statement* sees a fresh snapshot.

```
Transaction A                          Transaction B
-------------                          -------------
BEGIN;
SELECT bal FROM accounts WHERE id=1;
-- reads 100
                                       BEGIN;
                                       UPDATE accounts SET bal=50 WHERE id=1;
                                       COMMIT;
SELECT bal FROM accounts WHERE id=1;
-- reads 50  (NON-REPEATABLE READ)
COMMIT;
```

**REPEATABLE READ** — prevents dirty + non-repeatable reads. The whole *transaction* sees one snapshot taken at its first read. In PostgreSQL this is true snapshot isolation, which **also prevents phantom reads** (stronger than the SQL standard requires).

```
Transaction A                          Transaction B
-------------                          -------------
BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT count(*) FROM orders WHERE amt>100;
-- reads 5
                                       BEGIN;
                                       INSERT INTO orders(amt) VALUES(200);
                                       COMMIT;
SELECT count(*) FROM orders WHERE amt>100;
-- still reads 5  (no phantom in PG)
COMMIT;
```
But it still allows the **serialization anomaly (write skew)**:

```
Transaction A                          Transaction B
-------------                          -------------
-- Rule: at least 1 doctor on call
BEGIN ISOLATION LEVEL REPEATABLE READ;  BEGIN ISOLATION LEVEL REPEATABLE READ;
SELECT count(*) FROM oncall WHERE on=true; -- 2
                                        SELECT count(*) FROM oncall WHERE on=true; -- 2
UPDATE oncall SET on=false WHERE id=1;
                                        UPDATE oncall SET on=false WHERE id=2;
COMMIT;                                 COMMIT;
-- Now ZERO doctors on call — invariant violated, yet each txn saw 2.
```

**SERIALIZABLE** — prevents all anomalies. PostgreSQL uses Serializable Snapshot Isolation (SSI): it tracks read/write dependencies and aborts one transaction with a serialization failure (`ERROR: could not serialize access`, SQLSTATE `40001`) if a dangerous cycle is detected. In the write-skew example above, one transaction would be rolled back and must retry.

**Anomaly matrix:**

| Isolation Level   | Dirty Read | Non-Repeatable Read | Phantom Read           | Serialization Anomaly |
|-------------------|------------|---------------------|------------------------|-----------------------|
| READ UNCOMMITTED  | Possible*  | Possible            | Possible               | Possible              |
| READ COMMITTED    | Prevented  | Possible            | Possible               | Possible              |
| REPEATABLE READ   | Prevented  | Prevented           | Prevented (in PG)†     | Possible              |
| SERIALIZABLE      | Prevented  | Prevented           | Prevented              | Prevented             |

\* In PostgreSQL, READ UNCOMMITTED behaves as READ COMMITTED, so dirty reads are Prevented there.
† The SQL standard allows phantoms at REPEATABLE READ; PostgreSQL's MVCC implementation prevents them.

**Setting isolation level in PostgreSQL:**

```sql
-- Per transaction:
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- ... statements ...
COMMIT;

-- Or after BEGIN, before any query:
BEGIN;
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Session default:
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

**Follow-up questions the interviewer will ask:**

1. *"Why does PostgreSQL prevent phantoms at REPEATABLE READ when the standard doesn't require it?"* — Because PG implements it as snapshot isolation: the entire transaction reads from one consistent snapshot, so newly inserted rows from other transactions are simply not visible. The standard was written around lock-based engines.

2. *"What must application code do under SERIALIZABLE?"* — Always wrap transactions in retry logic. SERIALIZABLE can abort with SQLSTATE 40001 at commit; the app must catch it and re-run the transaction.

3. *"What's the cost difference between READ COMMITTED and SERIALIZABLE?"* — SERIALIZABLE adds SIReadLock tracking (predicate locks) overhead and causes retries under contention. READ COMMITTED is cheapest but pushes correctness burden onto the developer (you must use `SELECT ... FOR UPDATE`, etc.).

**Common mistakes that get you rejected:**
- Claiming PostgreSQL allows phantom reads at REPEATABLE READ. It doesn't.
- Forgetting PostgreSQL maps READ UNCOMMITTED to READ COMMITTED.
- Not knowing SERIALIZABLE requires application-side retry on 40001.
- Confusing non-repeatable read (same row changes) with phantom read (row set changes).

---

### Q43: Normalize a Table from Un-normalized Through BCNF
**Company:** Infosys, TCS, Wipro, Cognizant, any OA  **Difficulty:** 🟡  **Frequency:** 🔥🔥🔥  **Round:** OA/Phone

**Question:** Take an un-normalized orders table and normalize it step by step through 1NF, 2NF, 3NF, and BCNF, explaining the functional dependencies at each step. Then explain when you'd intentionally denormalize.

**What interviewer is testing:** Whether you understand *why* each normal form exists (eliminating a specific kind of anomaly), not just memorized definitions.

**Ideal Answer:**

**Un-normalized table** — repeating groups, multiple values per cell:

```
orders_unf
+---------+-----------+------------------------+-------------------+----------+
| order_id| cust_name | products               | cust_city         | city_zip |
+---------+-----------+------------------------+-------------------+----------+
| 1       | Alice     | Pen:2, Notebook:1      | Pune              | 411001   |
| 2       | Bob       | Pen:5                  | Mumbai            | 400001   |
+---------+-----------+------------------------+-------------------+----------+
```
Problems: `products` holds repeating groups; can't query "how many pens sold" easily.

**1NF — atomic values, no repeating groups.** Each cell holds a single value; one row per product line.

```sql
CREATE TABLE orders_1nf (
    order_id   INT,
    cust_name  TEXT,
    cust_city  TEXT,
    city_zip   TEXT,
    product    TEXT,
    qty        INT,
    PRIMARY KEY (order_id, product)
);
```
FDs present (problematic):
- `order_id → cust_name, cust_city, city_zip` (partial — depends on only part of the PK)
- `cust_city → city_zip` (transitive)

**2NF — 1NF + no partial dependencies on a composite key.** Every non-key column must depend on the *whole* PK. `cust_name`/`cust_city` depend only on `order_id`, not on `product`, so split them out.

```sql
CREATE TABLE orders_2nf (
    order_id   INT PRIMARY KEY,
    cust_name  TEXT,
    cust_city  TEXT,
    city_zip   TEXT
);

CREATE TABLE order_items_2nf (
    order_id   INT REFERENCES orders_2nf(order_id),
    product    TEXT,
    qty        INT,
    PRIMARY KEY (order_id, product)
);
```
Remaining problem in `orders_2nf`: `cust_city → city_zip` is a transitive dependency (`order_id → cust_city → city_zip`).

**3NF — 2NF + no transitive dependencies.** Non-key columns must not determine other non-key columns. Extract the city→zip relationship.

```sql
CREATE TABLE orders_3nf (
    order_id   INT PRIMARY KEY,
    cust_name  TEXT,
    cust_city  TEXT REFERENCES cities(cust_city)
);

CREATE TABLE cities (
    cust_city  TEXT PRIMARY KEY,
    city_zip   TEXT
);

CREATE TABLE order_items (
    order_id   INT REFERENCES orders_3nf(order_id),
    product    TEXT,
    qty        INT,
    PRIMARY KEY (order_id, product)
);
```

**BCNF — every determinant must be a candidate key.** 3NF still allows an anomaly when a non-prime attribute determines part of a candidate key. Classic example: a `course_instructor` table.

```
Rule: each course has one instructor per subject; each instructor teaches one subject.
FDs:  (student, subject) → instructor      [candidate key: (student, subject)]
      instructor → subject                 [instructor determines subject, but is NOT a key]
```
`instructor → subject` violates BCNF because `instructor` is not a candidate key. Decompose:

```sql
CREATE TABLE instructor_subject (
    instructor TEXT PRIMARY KEY,
    subject    TEXT
);

CREATE TABLE student_instructor (
    student    TEXT,
    instructor TEXT REFERENCES instructor_subject(instructor),
    PRIMARY KEY (student, instructor)
);
```
Now every determinant is a candidate key — BCNF satisfied.

**When to INTENTIONALLY denormalize:**
- **Reporting / analytics tables:** A nightly-built wide "fact" table avoids 6-way JOINs on every dashboard load.
- **Read-heavy paths:** Storing `cust_city` redundantly on `orders` to skip a JOIN on a hot endpoint that runs millions of times.
- **Avoiding expensive JOINs:** Pre-computed aggregates (e.g., `order_total` cached on the orders row) when recomputing from line items is costly.
- **Trade-off:** You trade write complexity and the risk of update anomalies (must keep copies in sync, often via triggers or app logic) for read speed. Denormalize *after* measuring, never preemptively.

```sql
-- Denormalized read model, refreshed periodically:
CREATE MATERIALIZED VIEW order_report AS
SELECT o.order_id, o.cust_name, c.city_zip,
       SUM(oi.qty) AS total_items
FROM orders_3nf o
JOIN cities c ON c.cust_city = o.cust_city
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY o.order_id, o.cust_name, c.city_zip;

REFRESH MATERIALIZED VIEW CONCURRENTLY order_report;
```

**Follow-up questions the interviewer will ask:**

1. *"Difference between 3NF and BCNF?"* — 3NF allows a non-prime attribute to be functionally dependent only if it doesn't create a transitive dependency, but it tolerates the case where a non-key determines part of a candidate key. BCNF is stricter: *every* determinant must be a candidate key. Most schemas in 3NF are already in BCNF; the difference only shows up with overlapping candidate keys.

2. *"Can BCNF decomposition lose information?"* — BCNF decomposition is always lossless-join, but it's not always dependency-preserving. Sometimes you can't enforce an FD without a JOIN, which is why people occasionally stop at 3NF (always achievable as both lossless and dependency-preserving).

3. *"What normal form do you target in practice?"* — 3NF for OLTP. Then selectively denormalize hot read paths with materialized views or cached columns.

**Common mistakes that get you rejected:**
- Jumping straight to "split into more tables" without naming the *dependency* being eliminated.
- Confusing 2NF (partial dependency) with 3NF (transitive dependency).
- Saying normalization is always good — production systems deliberately denormalize for performance.

---

### Q44: Explain B-tree Index Internals and When PostgreSQL Skips the Index
**Company:** Uber, Zomato, Swiggy  **Difficulty:** 🔴  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Describe the structure of a B-tree index in PostgreSQL — root, branch, leaf pages — and explain when the planner chooses *not* to use an available index.

**What interviewer is testing:** Whether you understand indexes as a physical data structure with real costs, not magic that "makes queries fast."

**Ideal Answer:**

**Structure (B-tree = balanced tree):**
- **Root page:** single top page; holds separator keys pointing down to branch pages.
- **Branch (internal) pages:** hold separator keys and pointers to lower levels. Used purely for navigation.
- **Leaf pages:** hold the actual index entries — the indexed key value plus a **heap pointer (TID/ctid)** identifying the physical row location in the table heap. The index does *not* store the full row (unless covering, see Q59).
- **Leaf pages are doubly linked** (sibling pointers both directions), enabling efficient ordered range scans (`BETWEEN`, `>`, `ORDER BY`) without going back up the tree.

```
                 [ Root ]
                /   |    \
          [Branch][Branch][Branch]      <- navigation only
           /  \      |       /  \
       [Leaf]<->[Leaf]<->[Leaf]<->[Leaf]   <- key + heap pointer, doubly linked
```

**Tree height:** Each page (~8KB) holds hundreds of keys, so fan-out is high. For ~1M rows, height is typically **3–4 levels**, meaning a point lookup is ~3–4 page reads. Even 100M+ rows usually stay at 4–5 levels — that's the power of high fan-out.

**A point lookup:** root → branch → leaf (find key + heap pointer) → fetch row from heap. A range scan: descend to the first matching leaf, then walk the sibling linked list.

**When PostgreSQL SKIPS the index (chooses Seq Scan instead):**
1. **Low selectivity** — query returns a large fraction of rows (rule of thumb >~5–10%). Random heap fetches per index entry become more expensive than one sequential pass.
2. **Small table** — if the table fits in a few pages, a sequential scan is cheaper than the index indirection.
3. **Leading column not in WHERE** — for a composite index `(a,b,c)`, a predicate on `b` alone can't use the index efficiently (left-prefix rule, Q45).
4. **Function/type mismatch** — `WHERE LOWER(email)=...` won't use a plain index on `email` (needs an expression index).
5. **Stale statistics** — planner misestimates row counts; fix with `ANALYZE`.

**EXPLAIN showing the difference:**

```sql
-- Highly selective: planner uses the index
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 12345;
-- Index Scan using idx_cust on orders  (cost=0.42..8.45 rows=3 width=64)
--   Index Cond: (customer_id = 12345)

-- Low selectivity: planner ignores the index
EXPLAIN ANALYZE SELECT * FROM orders WHERE status = 'completed';  -- 80% of rows
-- Seq Scan on orders  (cost=0.00..18334.00 rows=800000 width=64)
--   Filter: (status = 'completed')
```

**Follow-up questions the interviewer will ask:**

1. *"Why is a Seq Scan sometimes faster than an Index Scan?"* — Index Scan does random I/O (jump to scattered heap pages, one per matched row); Seq Scan does sequential I/O (read pages in order, prefetch-friendly). When many rows match, sequential beats scattered random access.

2. *"What's a Bitmap Index Scan?"* — A middle ground: PostgreSQL scans the index to build a bitmap of heap pages, sorts them, then reads the heap in physical order. Used when too many rows for a plain Index Scan but too few to justify a full Seq Scan.

3. *"How does the planner decide selectivity?"* — From `pg_stats` (histograms, n_distinct, most-common-values) gathered by `ANALYZE`. Bad stats → bad plans.

**Common mistakes that get you rejected:**
- Saying "an index always makes a query faster." It can be slower for low-selectivity queries.
- Thinking leaf pages store the full row (they store key + heap pointer).
- Not knowing about Bitmap Index Scan as the in-between option.

---

### Q45: Composite Index Left-Prefix Rule and Column Ordering
**Company:** Amazon, Flipkart, Razorpay  **Difficulty:** 🟡  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** Given `CREATE INDEX idx ON orders (customer_id, order_date, status)`, explain which queries can use the index and which can't, and how you'd decide column order.

**What interviewer is testing:** Practical indexing intuition. The left-prefix rule is the single most common indexing mistake in production.

**Ideal Answer:**

**The index:**
```sql
CREATE INDEX idx ON orders (customer_id, order_date, status);
```
A composite B-tree is sorted by `customer_id` first, then `order_date` within each customer, then `status`. You can only use a contiguous **left prefix** of the columns.

**Queries that USE the index:**
```sql
-- Uses (customer_id) — leading column
WHERE customer_id = 100;

-- Uses (customer_id, order_date)
WHERE customer_id = 100 AND order_date = '2026-01-01';

-- Uses all three
WHERE customer_id = 100 AND order_date = '2026-01-01' AND status = 'paid';

-- Uses (customer_id) for seek + (order_date) for range
WHERE customer_id = 100 AND order_date >= '2026-01-01';
```

**Queries that SKIP (or only partially use) the index:**
```sql
-- SKIP: order_date is not the leading column
WHERE order_date = '2026-01-01';

-- SKIP: status alone, no leading column
WHERE status = 'paid';

-- PARTIAL: uses customer_id, but status can't be used as an index condition
-- because order_date (the middle column) is a RANGE, not equality.
WHERE customer_id = 100 AND order_date >= '2026-01-01' AND status = 'paid';
--   -> index seeks on (customer_id, order_date range); status applied as a filter, not a seek
```

**Equality-before-range rule:** Columns used with `=` should come *before* columns used with a range (`>`, `<`, `BETWEEN`). Once the planner hits a range column, it can no longer use subsequent columns for index seeks — only as filters. So an index on `(customer_id, status, order_date)` would be better if your query is `customer_id = ? AND status = ? AND order_date >= ?`.

**Column-order strategy / the trade-off:**
- It is **not** simply "highest cardinality first." That's a common myth.
- Order should be driven by **query shape**: put **equality-predicate columns first**, then the **range column**, then columns only needed for ordering or covering.
- Among equality columns, the leading one should be one that appears in (almost) all your queries — otherwise those queries can't use the index at all.
- Cardinality matters as a tiebreaker: a more selective leading column narrows the search faster, but selectivity is useless if queries don't filter on that column.

```sql
-- For query: WHERE customer_id = ? AND status = ? AND order_date >= ?
-- BEST order: equality columns first, range last
CREATE INDEX idx_better ON orders (customer_id, status, order_date);
```

**Follow-up questions the interviewer will ask:**

1. *"Can an index on (a,b,c) satisfy `WHERE a=? AND c=?`?"* — It uses `a` for the seek; `c` becomes a filter on the rows matching `a` (it cannot skip `b`). Unless `b` has few distinct values where PG can use a "skip scan"-like access — but classic PostgreSQL B-tree treats `c` as a filter here.

2. *"Does column order matter for an index used only for ORDER BY?"* — Yes. `(customer_id, order_date)` supports `WHERE customer_id=? ORDER BY order_date` without a sort. The direction (ASC/DESC) and `NULLS FIRST/LAST` must also match or be reversible.

3. *"Why not just create three single-column indexes?"* — The planner can combine them with a BitmapAnd, but that's slower than one composite index for multi-column predicates, and costs more write overhead. A composite index is usually better when queries filter on the columns together.

**Common mistakes that get you rejected:**
- Claiming `WHERE status = ?` uses an index that leads with `customer_id`.
- Blindly saying "highest cardinality column first" — order should follow query predicates (equality before range).
- Forgetting that a range column kills index usability for everything after it.

---

### Q46: Compare Hash Index vs B-tree vs LSM-tree
**Company:** Cassandra teams, Redis-heavy startups  **Difficulty:** 🔴  **Frequency:** 🔥  **Round:** Onsite

**Question:** Compare B-tree, hash index, and LSM-tree storage structures. When would you pick each?

**What interviewer is testing:** Storage-engine depth and matching data structures to access patterns — a system-design-flavored question.

**Ideal Answer:**

**B-tree:**
- Balanced tree, sorted leaves. Supports `=`, `<`, `>`, `BETWEEN`, `ORDER BY`, prefix matches.
- Updates are done in place (within page; may cause page splits).
- Read and write are both `O(log n)`. The default and general-purpose index.
- Best for: range queries, sorted scans, mixed read/write OLTP.

**Hash index:**
- Maps a hash of the key to a bucket; `O(1)` average for **exact-match** lookups.
- **No range support, no ordering** — useless for `<`, `>`, `BETWEEN`, `ORDER BY`.
- In PostgreSQL: hash indexes were **not WAL-logged (not crash-safe / not replicated) until PG 10**. Since PG 10 they're WAL-logged and usable, but rarely beat B-tree enough to matter.
- Best for: very large keys where only equality is ever needed (the hash is smaller than the full key).

```sql
CREATE INDEX idx_token_hash ON sessions USING hash (session_token);
-- Only helps: WHERE session_token = '...';  NOT range queries.
```

**LSM-tree (Log-Structured Merge tree):**
- Writes go to an in-memory **memtable**, flushed sequentially to immutable **SSTables** on disk; background **compaction** merges and discards stale versions.
- **Write-optimized:** writes are sequential appends → extremely high write throughput. Used by Cassandra, RocksDB, LevelDB, ScyllaDB.
- Costs: **compaction overhead** (CPU/IO churn) and **read amplification** (a read may check the memtable + several SSTables + bloom filters before finding the value).
- Best for: write-heavy and append-heavy workloads, time-series ingestion, event logs.

**How to pick:**

| Workload | Pick | Why |
|---|---|---|
| Time-series / bulk inserts, write-heavy | **LSM-tree** | Sequential writes, high ingest throughput |
| Exact-match cache / session lookup | **Hash** | O(1) point lookups, no range needed |
| Range analytics, ORDER BY, mixed OLTP | **B-tree** | Sorted, supports ranges and ordering |

**Follow-up questions the interviewer will ask:**

1. *"What is write amplification vs read amplification in an LSM-tree?"* — Write amplification: a single logical write gets rewritten multiple times during compaction (memtable → L0 → L1 ...). Read amplification: a single read may inspect multiple SSTables across levels (mitigated by bloom filters and per-SSTable indexes).

2. *"Why is PostgreSQL's default not a hash index?"* — B-tree handles both equality and range, and `=` lookups on a B-tree are already fast (~3–4 page reads). Hash's narrow benefit rarely justifies losing range/order support.

3. *"How do LSM-trees handle deletes?"* — With **tombstones**: a delete writes a marker; the actual data is removed only during compaction. Tombstones can cause read slowdowns and the "tombstone problem" in Cassandra if not compacted.

**Common mistakes that get you rejected:**
- Saying a hash index supports range queries — it never does.
- Not knowing LSM-trees trade read performance for write performance.
- Recommending hash indexes in PostgreSQL without mentioning the pre-PG10 durability caveat.

---

### Q47: Explain MVCC in PostgreSQL in Depth
**Company:** Amazon, Google, Uber  **Difficulty:** ⚫  **Frequency:** 🔥🔥  **Round:** Onsite senior

**Question:** Explain how MVCC works in PostgreSQL — tuple visibility, dead tuples, VACUUM, transaction ID wraparound, and HOT updates.

**What interviewer is testing:** Senior/staff-level engine internals. This is where strong candidates separate themselves.

**Ideal Answer:**

**Core idea:** Instead of locking rows for readers, PostgreSQL keeps **multiple versions** of each row. Readers and writers don't block each other; each transaction sees a consistent snapshot.

**Per-tuple header fields:**
- **`xmin`** — the transaction ID (XID) that *created* (inserted) this row version.
- **`xmax`** — the transaction ID that *deleted* or *updated* (superseded) this row version. `0`/invalid if still live.

**UPDATE behavior:** An `UPDATE` does **not** overwrite. It marks the old tuple's `xmax` and inserts a **new tuple version** with a fresh `xmin`. The old version stays in the heap as a **dead tuple** until VACUUM reclaims it.

**Snapshot visibility rule (simplified):** A transaction sees a row version if:
- its `xmin` is committed and `<=` the transaction's snapshot (and not in the snapshot's in-progress list), AND
- its `xmax` is invalid, OR the deleting transaction is not committed / is after the snapshot.

**ASCII timeline — two concurrent transactions:**

```
Initial: row R = (id=1, bal=100), xmin=50, xmax=0   (created by txn 50, live)

Time   Txn A (XID 100, READ COMMITTED)        Txn B (XID 101)
----   --------------------------------       --------------------------------
t1     BEGIN; snapshot taken
t2     SELECT bal WHERE id=1  -> 100
       (sees version xmin=50, xmax=0)
t3                                             BEGIN;
t4                                             UPDATE R SET bal=200 WHERE id=1;
                                               -> old tuple: xmin=50, xmax=101 (dead-pending)
                                               -> new tuple: xmin=101, xmax=0
t5     SELECT bal WHERE id=1  -> 100           (B not committed; A still sees old)
t6                                             COMMIT;
t7     SELECT bal WHERE id=1  -> 200           (new statement = new snapshot in RC; B committed)
t8     COMMIT;
```
Under REPEATABLE READ, A's t7 read would still return 100 (snapshot fixed at BEGIN).

**Dead tuples and VACUUM:**
- Every UPDATE/DELETE leaves dead tuples → **table bloat**.
- **VACUUM** removes dead tuples no longer visible to any transaction, marks space reusable, and updates the **visibility map** (pages where all tuples are visible to everyone — enables index-only scans and lets future vacuums skip them).
- **autovacuum** runs this automatically based on dead-tuple thresholds.
- `VACUUM FULL` rewrites the whole table to physically shrink it (takes an exclusive lock — avoid on live tables; use `pg_repack` instead).

**Transaction ID wraparound:**
- XIDs are **32-bit** → only ~**2 billion** usable values, after which they wrap around.
- If old tuples aren't "frozen," wraparound could make committed rows suddenly look like they're from the future and become invisible — catastrophic data loss.
- **autovacuum freezes** old tuples (sets a special FrozenXID / sets the frozen bit) so they're always visible regardless of XID counter, preventing wraparound. If autovacuum can't keep up, PostgreSQL forces an emergency vacuum and eventually refuses new transactions to protect data.

**HOT (Heap-Only Tuple) updates:**
- Optimization: if an UPDATE changes only **non-indexed columns** AND the new version fits on the **same heap page**, PostgreSQL chains the new tuple to the old one within the page and **does not** add new entries to any index.
- Benefits: less index bloat, less WAL, faster updates. The old version becomes reclaimable by a lightweight page-level cleanup.
- Defeated when you update an indexed column or the page is full.

**Inspecting it:**
```sql
SELECT xmin, xmax, ctid, * FROM accounts WHERE id = 1;
-- ctid = (page, offset) physical location; changes after a non-HOT update.

SELECT relname, n_dead_tup, n_live_tup, last_autovacuum
FROM pg_stat_user_tables WHERE relname = 'accounts';
```

**Follow-up questions the interviewer will ask:**

1. *"Why can a DELETE-heavy table slow down even after deleting rows?"* — Dead tuples remain until VACUUM runs; sequential scans and indexes still traverse the bloat. You need VACUUM (and possibly pg_repack) to reclaim space.

2. *"How does MVCC let readers not block writers?"* — Writers create new versions instead of mutating in place, so a reader's snapshot still points to the old, still-valid version. No shared read locks needed.

3. *"What's the downside of MVCC vs in-place update engines?"* — Bloat and vacuum overhead. PostgreSQL must clean up old versions; a poorly tuned autovacuum leads to bloat and wraparound risk.

**Common mistakes that get you rejected:**
- Saying UPDATE overwrites the row in place — it creates a new version.
- Not knowing what VACUUM does or why dead tuples exist.
- Being unaware of XID wraparound (a classic senior-level gotcha).

---

### Q48: Walk Through a Systematic Query Optimization Workflow
**Company:** Any SDE-2 onsite  **Difficulty:** 🔴  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** A query is slow. Walk me through your systematic process to diagnose and fix it.

**What interviewer is testing:** Whether you have a *repeatable methodology* instead of randomly adding indexes and hoping.

**Ideal Answer:**

**The checklist:**

1. **`EXPLAIN ANALYZE` the query** — get the real plan with actual times and row counts (not just `EXPLAIN`, which only estimates). Add `BUFFERS` to see I/O.
2. **Find the most expensive node** — look for the highest `actual time` and large gaps between `estimated rows` and `actual rows` (estimation errors signal stale stats).
3. **Check statistics (`pg_stats`)** — are `n_distinct`, MCVs, histograms current? Run `ANALYZE` if stale.
4. **Check for missing indexes** — is there a Seq Scan on a large table with a selective filter? A Sort that an index could avoid?
5. **Check join order & method** — Nested Loop over a large unindexed inner table is a red flag (see Q49).
6. **Check functions in WHERE** — `WHERE LOWER(email)=...` or `WHERE date_col::text=...` defeat plain indexes; needs an expression index or rewrite.
7. **`VACUUM ANALYZE`** — clears dead-tuple bloat and refreshes stats.
8. **Rewrite the query** — replace a correlated subquery with a JOIN, push filters down, avoid `SELECT *`, use `EXISTS` instead of `IN` for large subqueries.

**Realistic slow query walkthrough:**

```sql
EXPLAIN (ANALYZE, BUFFERS)
SELECT o.order_id, o.total
FROM orders o
WHERE LOWER(o.email) = 'alice@example.com'
  AND o.created_at >= '2026-01-01'
ORDER BY o.created_at DESC
LIMIT 20;
```
Suppose the plan shows:
```
Limit  (actual time=812.4..812.5 rows=20)
  ->  Sort  (actual time=812.4..812.4 rows=20)  Sort Method: top-N heapsort
        ->  Seq Scan on orders  (actual time=0.1..790.2 rows=18 loops=1)
              Filter: ((lower(email) = 'alice@example.com') AND (created_at >= '2026-01-01'))
              Rows Removed by Filter: 4999982
```
**Diagnosis:**
- Most expensive node: `Seq Scan` scanning ~5M rows to return 18.
- `LOWER(email)` defeats any plain index on `email`.
- No usable index on `created_at` for the filter/sort either.

**Fix:**
```sql
-- Expression index for case-insensitive equality:
CREATE INDEX idx_orders_lower_email ON orders (LOWER(email));

-- If many queries filter by email then sort by date, a composite expression index:
CREATE INDEX idx_orders_email_created ON orders (LOWER(email), created_at DESC);
```
**Re-measure:**
```
Limit  (actual time=0.05..0.07 rows=20)
  ->  Index Scan using idx_orders_email_created on orders (actual time=0.04..0.06 rows=18)
        Index Cond: (lower(email) = 'alice@example.com' AND created_at >= '2026-01-01')
```
812ms → <1ms. The Index Scan also returns rows pre-sorted by `created_at DESC`, so the Sort node disappears entirely.

**Follow-up questions the interviewer will ask:**

1. *"Difference between EXPLAIN and EXPLAIN ANALYZE?"* — `EXPLAIN` shows the planner's *estimated* plan without running it. `EXPLAIN ANALYZE` actually *executes* it and reports real timings and row counts (be careful — it runs the query, including writes unless wrapped in a rolled-back transaction).

2. *"What does a big gap between estimated and actual rows tell you?"* — Stale/insufficient statistics; the planner is misestimating and may pick a bad join method. Fix with `ANALYZE` or increase `default_statistics_target` for skewed columns.

3. *"When does adding an index NOT help?"* — Low-selectivity filters, tiny tables, or when the bottleneck is a Sort/Hash that spills to disk (fix with `work_mem`) rather than a scan.

**Common mistakes that get you rejected:**
- Adding indexes by guesswork without reading `EXPLAIN ANALYZE`.
- Using `EXPLAIN` (estimates) when you needed `EXPLAIN ANALYZE` (reality).
- Ignoring the estimate-vs-actual row gap, which is the #1 signal of a bad plan.

---

### Q49: Explain JOIN Algorithms and When the Planner Picks Each
**Company:** Amazon, Flipkart, Zomato  **Difficulty:** 🟡  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Explain Nested Loop, Hash Join, and Merge Join. When does the PostgreSQL planner choose each?

**What interviewer is testing:** Understanding of physical join execution and the cost model behind plan choices.

**Ideal Answer:**

**Nested Loop Join:**
- For each row in the outer table, scan the inner table for matches. Complexity `O(N × M)` — but `O(N × log M)` if the inner side is indexed.
- Best when the **outer side is small** and the **inner side is indexed** (so each inner lookup is cheap), or for small result sets.
- Cheap startup cost; bad if both sides are large and unindexed.

**Hash Join:**
- **Build phase:** create an in-memory hash table on the **smaller** input keyed on the join column. **Probe phase:** scan the larger input, hashing each row to find matches. Complexity `O(N + M)`.
- Best for **large, unsorted** tables joined on **equality** (`=` only).
- Requires memory (`work_mem`); if the hash doesn't fit, it spills to disk in batches.

**Merge Join:**
- Both inputs are **sorted** on the join key, then merged in a single linear pass like a zipper. Complexity `O(N + M)` if already sorted, else `O(N log N + M log M)` to sort first.
- Best when **both sides are already sorted** — e.g., both have a usable index on the join column, or output is needed in sorted order anyway. Great for large datasets and range/merge-friendly joins.

**Summary:**

| Algorithm | Complexity | Best when |
|---|---|---|
| Nested Loop | O(N×M), O(N·logM) if indexed | Small outer, indexed inner; small results |
| Hash Join | O(N+M) | Large unsorted tables, equality join, enough memory |
| Merge Join | O(N+M) if sorted | Both sides already sorted / indexed on join key |

**Forcing a join type for testing (never in production):**
```sql
-- See what the planner does without hash joins:
SET enable_hashjoin = off;
EXPLAIN ANALYZE
SELECT * FROM orders o JOIN customers c ON c.id = o.customer_id;

-- Reset:
SET enable_hashjoin = on;
-- Similar toggles: enable_nestloop, enable_mergejoin
```

**Follow-up questions the interviewer will ask:**

1. *"You see a Nested Loop in a slow query over two large tables. What's likely wrong?"* — The planner underestimated the row count (stale stats) and assumed a small outer set, or the inner table lacks an index. Run `ANALYZE`; add the inner index; consider whether a Hash Join is more appropriate.

2. *"Why does the planner build the hash on the smaller table?"* — To minimize memory and keep the hash table cache-friendly; the larger table just streams through as the probe side.

3. *"When is a Merge Join chosen even though sorting is expensive?"* — When indexes already provide sorted input (no sort needed), or when the query's `ORDER BY` requires that sort anyway, so the merge order is "free."

**Common mistakes that get you rejected:**
- Saying Hash Join supports range joins — it's equality-only.
- Not knowing Nested Loop is fine (even optimal) when the inner side is indexed.
- Forgetting Hash Join needs memory and can spill to disk.

---

### Q50: Demonstrate a Deadlock, Its Detection, and Resolution
**Company:** PayPal, PhonePe, Razorpay  **Difficulty:** 🔴  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Show two transactions that deadlock. Explain how PostgreSQL detects and resolves it, and how to prevent deadlocks.

**What interviewer is testing:** Concurrency control under contention and practical prevention strategies — critical for payments systems.

**Ideal Answer:**

**A deadlock:** two transactions each hold a lock the other needs, forming a cycle. Neither can proceed.

```sql
-- T1                                  -- T2
BEGIN;                                 BEGIN;
UPDATE accounts SET bal=bal-100        UPDATE accounts SET bal=bal-50
  WHERE id='A';   -- locks row A         WHERE id='B';   -- locks row B

-- T1 now wants B:                      -- T2 now wants A:
UPDATE accounts SET bal=bal+100        UPDATE accounts SET bal=bal+50
  WHERE id='B';   -- BLOCKS (T2 holds B)  WHERE id='A';   -- BLOCKS (T1 holds A)
-- ... cycle: T1 waits for T2, T2 waits for T1 ...
```
T1 locked A then waits for B; T2 locked B then waits for A. Classic cycle.

**Detection:**
- PostgreSQL maintains a **waits-for graph** (who is waiting on whom).
- After a lock wait exceeds **`deadlock_timeout`** (default **1 second**), it runs **cycle detection** on the graph.
- If a cycle is found, PostgreSQL **chooses one transaction as the victim**, aborts it, and raises:
  ```
  ERROR:  deadlock detected
  DETAIL:  Process 123 waits for ShareLock on transaction 456; ...
  SQLSTATE: 40P01
  ```
- The victim rolls back; the other transaction proceeds. The application must catch `40P01` and retry.

**Prevention:**
1. **Consistent lock ordering** — always acquire locks in a deterministic order (e.g., always lock the lower account id first). This makes a cycle impossible.
   ```sql
   -- Both transactions lock the smaller id first:
   SELECT * FROM accounts WHERE id IN ('A','B') ORDER BY id FOR UPDATE;
   -- Now do the updates.
   ```
2. **Keep transactions short** — hold locks for as little time as possible.
3. **`SELECT ... FOR UPDATE SKIP LOCKED`** for queue/worker patterns — workers grab available rows and skip locked ones instead of blocking, eliminating contention entirely.
   ```sql
   SELECT * FROM jobs
   WHERE status='pending'
   ORDER BY created_at
   FOR UPDATE SKIP LOCKED
   LIMIT 1;
   ```
4. **Retry logic** — even with discipline, treat `40P01` as a transient error and retry with backoff.

**Follow-up questions the interviewer will ask:**

1. *"How does PostgreSQL pick the victim?"* — It generally aborts the transaction that would be cheaper/simpler to roll back (often the one that detected the deadlock / is later in the cycle). You can't reliably predict it, so always retry.

2. *"Difference between a deadlock and a lock wait?"* — A lock wait is one transaction blocking on another and eventually proceeding. A deadlock is a *cycle* where neither can ever proceed; only then does PG abort a victim. Lock waits that never form a cycle just wait (subject to `lock_timeout` if set).

3. *"How does SKIP LOCKED help queues?"* — Multiple consumers polling the same table won't block each other; each grabs a different unlocked row, giving lock-free work distribution.

**Common mistakes that get you rejected:**
- Saying PostgreSQL prevents deadlocks automatically — it *detects and resolves* them, it doesn't prevent them.
- Forgetting that the application must catch 40P01 and retry.
- Not mentioning consistent lock ordering as the primary prevention.

---

### Q51: Explain Connection Pooling, HikariCP, and PgBouncer
**Company:** Swiggy, Zomato, any high-traffic startup  **Difficulty:** 🟡  **Frequency:** 🔥🔥  **Round:** Phone/Onsite

**Question:** Why is opening a PostgreSQL connection expensive? Explain connection pooling, key HikariCP parameters, PgBouncer modes, and pool exhaustion.

**What interviewer is testing:** Operational/production knowledge — the kind of thing that bites real services under load.

**Ideal Answer:**

**Why connections are expensive in PostgreSQL:**
- PostgreSQL uses a **process-per-connection** model — each connection forks a **separate OS process** (not a thread), costing ~**5–10MB RAM** each plus startup/auth/TLS handshake.
- Many connections → high memory use, more **context switching**, and contention on shared structures.
- So you keep a **pool** of warm, reusable connections instead of opening/closing per request.

**HikariCP key parameters** (the standard Java/Spring pool):
- **`maximumPoolSize`** — max connections in the pool. The most important setting; sized to `≈ (core_count × 2) + effective_spindle_count`, not "as high as possible."
- **`minimumIdle`** — minimum idle connections kept ready (often set equal to max for steady load).
- **`connectionTimeout`** — how long a thread waits for a connection before throwing (default 30s). This is what surfaces during exhaustion.
- **`idleTimeout`** — how long an idle connection lives before being retired.
- **`maxLifetime`** — max age of any connection (should be a bit less than the DB/infra's connection timeout, e.g., < `wait_timeout`), to recycle connections proactively.

**PgBouncer modes** (lightweight external pooler in front of PostgreSQL):
- **Session pooling** — a server connection is assigned to a client for the whole session (until disconnect). Safest, lowest multiplexing.
- **Transaction pooling** — a server connection is assigned only for the duration of a transaction, then returned. **Recommended** — best multiplexing for web apps. Caveat: you can't use session-level features (prepared statements across transactions, `SET` session state, advisory session locks) without care.
- **Statement pooling** — connection returned after each statement; most aggressive, forbids multi-statement transactions.

**Pool exhaustion — symptoms and fixes:**
- *Symptoms:* requests hang then fail with connection-timeout errors; latency spikes; `connectionTimeout` exceptions; threads piling up waiting for a connection.
- *Causes:* connection leaks (not closing/returning), long-running transactions holding connections, pool too small for concurrency, or slow queries hogging connections.
- *Fixes:* fix leaks (use try-with-resources / framework-managed connections), shorten transactions, add PgBouncer transaction pooling to multiplex, right-size `maximumPoolSize`, set `leakDetectionThreshold` in HikariCP to log leaks.

**Follow-up questions the interviewer will ask:**

1. *"Why not just set maximumPoolSize to 1000?"* — More connections than the DB can usefully run in parallel causes context-switch thrashing and memory pressure, *reducing* throughput. The DB has limited CPU/IO; queue work in the pool instead of overwhelming the DB.

2. *"When would transaction pooling break your app?"* — When code relies on session state across transactions: server-side prepared statements, session-level `SET`, `LISTEN/NOTIFY`, session advisory locks. Those need session pooling or app changes.

3. *"You have 50 app instances × 20 connections = 1000 connections to one Postgres. Problem?"* — Likely too many for one Postgres. Put PgBouncer in front (transaction mode) so 1000 client connections multiplex onto, say, 50–100 real server connections.

**Common mistakes that get you rejected:**
- Thinking PostgreSQL connections are cheap threads — they're OS processes.
- "Just increase the pool size" as the universal fix — often makes it worse.
- Not knowing transaction-mode PgBouncer breaks session-scoped features.

---

### Q52: Explain PostgreSQL Replication, Sync vs Async, and Replication Lag
**Company:** Amazon, Uber, large-scale startups  **Difficulty:** 🔴  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** How does streaming replication work in PostgreSQL? Compare synchronous vs asynchronous replication and explain replication lag and stale reads.

**What interviewer is testing:** Understanding of HA architecture and the durability/latency/consistency trade-offs in distributed reads.

**Ideal Answer:**

**WAL streaming replication:**
- Every change is first written to the **WAL** (Write-Ahead Log) on the primary.
- A **standby/replica** connects to the primary and **streams the WAL records**, then **replays** them to stay in sync. This is physical replication (byte-level WAL).
- Standbys can serve **read-only** queries ("hot standby"), offloading reads from the primary.

**Asynchronous replication (default):**
- The primary **does not wait** for the standby. It acknowledges `COMMIT` as soon as its own WAL is flushed locally.
- *Pro:* low commit latency.
- *Con:* on primary failure/failover, the **last N transactions** that hadn't reached the standby are **lost** (non-zero RPO).

**Synchronous replication:**
- The primary **waits for the standby to confirm** it has written (or applied) the WAL before acknowledging `COMMIT`. Configured via `synchronous_standby_names` and `synchronous_commit` (`remote_write`, `on`, `remote_apply`).
- *Pro:* **zero data loss** for the synced standby (RPO ≈ 0).
- *Con:* **higher commit latency** (each commit waits a network round-trip); if the sync standby is down, commits can stall (mitigate with multiple sync candidates / quorum).

**Replication lag:**
- The delay between a change committing on the primary and being applied on a standby. Monitor via **`pg_stat_replication`** on the primary:
  ```sql
  SELECT client_addr, state,
         write_lag, flush_lag, replay_lag,
         sent_lsn, replay_lsn
  FROM pg_stat_replication;
  ```
  - **`write_lag`** — time until standby *wrote* the WAL.
  - **`flush_lag`** — time until standby *flushed* WAL to disk (durability point).
  - **`replay_lag`** — time until standby *applied* WAL so the change is *visible to readers*.

**Stale reads:**
- If you route reads to a replica that hasn't replayed the latest WAL, a user may **write to the primary then immediately read stale data** from the lagging replica ("read-your-writes" violation).
- *Mitigations:* route read-after-write to the primary; wait until the replica's `replay_lsn` >= the write's LSN; use synchronous `remote_apply`; or use sticky/session consistency at the app layer.

**Follow-up questions the interviewer will ask:**

1. *"Difference between flush_lag and replay_lag, and which matters for stale reads?"* — `flush_lag` is about durability (WAL safely on standby disk). `replay_lag` is about *visibility* — the standby may have the WAL but not yet applied it, so reads are stale until replay catches up. For read consistency, `replay_lag` is what matters.

2. *"How do you get zero data loss without paying full sync latency on every commit?"* — Use synchronous replication with `synchronous_commit = remote_write` (wait for WAL received, not applied), or a quorum of multiple sync standbys so any one being slow doesn't stall commits.

3. *"What is RPO vs RTO here?"* — RPO (Recovery Point Objective): how much data you can lose — async > 0, sync ≈ 0. RTO (Recovery Time Objective): how long failover takes to restore service.

**Common mistakes that get you rejected:**
- Saying async replication is "good enough" without acknowledging the data-loss-on-failover risk.
- Confusing flush_lag (durability) with replay_lag (read visibility).
- Not knowing replicas can serve stale reads and how to mitigate it.

---

### Q53: Compare Sharding Strategies in Depth
**Company:** Amazon, Flipkart, Razorpay  **Difficulty:** ⚫  **Frequency:** 🔥🔥  **Round:** Onsite/System Design

**Question:** Compare range, hash, consistent-hash, and directory-based sharding. How do cross-shard joins and distributed transactions work? Show how a query routes.

**What interviewer is testing:** Distributed data architecture at scale — a staff-level system design topic.

**Ideal Answer:**

**Sharding** = horizontally partitioning data across multiple independent database nodes (shards), each holding a subset, to scale beyond one machine.

**1. Range sharding** — partition by key ranges.
- e.g., `user_id 0–1M → shard1`, `1M–2M → shard2`.
- *Pro:* range queries (`WHERE user_id BETWEEN ...`) stay on one shard; simple to reason about.
- *Con:* **hotspots** — if `user_id` is monotonically increasing, all *new* users land on the last shard, overloading it while others sit idle.

**2. Hash sharding** — `shard = hash(key) % N`.
- *Pro:* **even distribution**, no hotspots from sequential keys.
- *Con:* range queries hit **all shards** (scatter-gather); **resharding is painful** — changing `N` with plain modulo remaps ~all keys, forcing a massive data move.

**3. Consistent hashing** — keys and nodes are placed on a hash **ring**; each key belongs to the next node clockwise. **Virtual nodes** smooth out distribution.
- *Pro:* adding/removing a node moves only **~K/N keys** (not nearly all of them) — far cheaper resharding. Used by **Cassandra, DynamoDB**.
- *Con:* more complex; range queries still scatter.

**4. Directory-based sharding** — a **lookup table/service** maps each entity (or key range) to a shard.
- *Pro:* maximum flexibility — rebalance by updating the directory; can move specific tenants.
- *Con:* the directory is an extra hop and a potential **single point of failure / bottleneck** (must be cached/replicated).

**Cross-shard JOINs:**
- A single SQL JOIN can't span shards. Options:
  - **Application-layer join** — query each shard, join results in app code.
  - **Scatter-gather** — broadcast to all shards, aggregate results.
  - **Denormalize / co-locate** related data on the same shard via a shared shard key (e.g., shard orders and order_items both by `customer_id`).

**Distributed transactions across shards:**
- **2PC** (Two-Phase Commit) for strong atomicity (costly, blocking — see Q60), or
- **Saga** pattern with compensating transactions for eventual consistency in microservices (see Q60).

**Query routing example — orders sharded by `customer_id`, 4 shards, hash sharding:**
```text
Query:   SELECT * FROM orders WHERE customer_id = 12345;

Router:  shard = hash(12345) % 4  ->  e.g., 1
         => route ONLY to shard1.   (single-shard query, fast)

Query:   SELECT * FROM orders WHERE total > 1000;   -- no shard key
Router:  no customer_id => SCATTER to shard0..3, GATHER + merge results.  (slow)
```
The lesson: choose a **shard key that matches your most common query predicate** so hot-path queries are single-shard.

**Follow-up questions the interviewer will ask:**

1. *"How do you pick a shard key?"* — Pick the column present in most queries (so they're single-shard) and with high enough cardinality and uniform distribution to spread load. For orders, `customer_id` co-locates a customer's data. Avoid low-cardinality or monotonically increasing keys.

2. *"How does consistent hashing reduce resharding cost vs modulo?"* — With `% N`, changing N remaps almost every key. On a hash ring, adding a node only steals the keys between it and its predecessor (~K/N), so only that fraction moves.

3. *"How do you keep globally unique IDs across shards?"* — Use UUIDs, or a distributed ID generator like Snowflake (timestamp + machine id + sequence), avoiding a single auto-increment bottleneck.

**Common mistakes that get you rejected:**
- Recommending `hash % N` without mentioning the resharding catastrophe (use consistent hashing).
- Forgetting range sharding causes hotspots with sequential keys.
- Saying cross-shard JOINs "just work" — they require app-side or scatter-gather handling.

---

### Q54: SQL vs NoSQL Decision Framework
**Company:** Any system design round  **Difficulty:** 🟡  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** How do you decide between SQL and the various NoSQL families (document, key-value, column-family, graph)?

**What interviewer is testing:** Whether you choose data stores by *access pattern and requirements*, not by hype.

**Ideal Answer:**

**Document (MongoDB, Couchbase):**
- Flexible/evolving schema, nested/semi-structured data, retrieved as whole documents.
- *Good for:* product catalogs, user profiles, content/CMS, config blobs.
- *Weak at:* complex multi-entity JOINs, strong cross-document transactions (improving but not Postgres-level).

**Key-value (Redis, DynamoDB):**
- `O(1)` lookup by key, extremely fast and simple.
- *Good for:* caching, session stores, rate limiting, feature flags, shopping carts by key.
- *Weak at:* querying by value / secondary attributes, relationships.

**Column-family / wide-column (Cassandra, HBase, ScyllaDB):**
- Write-optimized (LSM-tree), wide rows, partitioned by key; **no JOINs**, query-first modeling.
- *Good for:* time-series, event logging, IoT, very high write throughput, massive scale.
- *Weak at:* ad-hoc queries, anything needing JOINs or strong transactional integrity.

**Graph (Neo4j, Amazon Neptune):**
- Nodes + edges; traversals are first-class and cheap.
- *Good for:* social networks, recommendation/relationship queries, fraud detection, knowledge graphs ("friends of friends," shortest path).
- *Weak at:* bulk analytical scans, simple tabular workloads.

**When to pick PostgreSQL (relational) over all of them:**
- You need **ACID transactions** and strong relational integrity (foreign keys, constraints).
- **Complex queries:** multi-table JOINs, aggregations, window functions, ad-hoc analytics.
- Data is naturally **relational and structured**, with a fairly stable schema.
- *Bonus:* PostgreSQL also does JSONB (document-style), full-text search, arrays, and key-value via `hstore` — it's often "good enough" to avoid adding a second datastore. Default to PostgreSQL until a specific access pattern demands otherwise.

**Decision heuristic:**
```text
Need ACID + JOINs + ad-hoc queries?        -> PostgreSQL (SQL)
Just key -> value, ultra low latency?       -> Redis / DynamoDB (KV)
Flexible nested docs, schema churn?          -> MongoDB (Document)
Massive write throughput / time-series?      -> Cassandra (Column-family)
Relationship traversal is the core query?    -> Neo4j (Graph)
```

**Follow-up questions the interviewer will ask:**

1. *"Can't PostgreSQL with JSONB replace MongoDB?"* — For many use cases yes — JSONB gives flexible schema with GIN indexes and ACID. MongoDB still wins for sharded horizontal scale-out and a document-native developer experience at very large scale, but JSONB avoids premature polyglot persistence.

2. *"Why no JOINs in Cassandra?"* — Its distributed, partition-per-node model makes cross-partition joins prohibitively expensive. You denormalize and model tables per query instead (query-first design, see Q56).

3. *"How do you pick when requirements are mixed?"* — Polyglot persistence: PostgreSQL as the source of truth, Redis for caching/sessions, Elasticsearch for search, etc. But add stores only when a clear access pattern justifies the operational cost.

**Common mistakes that get you rejected:**
- "NoSQL scales, SQL doesn't" — false; PostgreSQL scales far, and NoSQL trades away consistency/queries.
- Picking a store by popularity instead of access pattern.
- Forgetting PostgreSQL's JSONB/full-text/array features can avoid a second database.

---

### Q55: Apply the CAP Theorem to Real Databases
**Company:** Amazon, Flipkart, any distributed systems round  **Difficulty:** 🔴  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** Explain the CAP theorem and classify PostgreSQL, Cassandra, DynamoDB, and Redis Cluster. What does PACELC add?

**What interviewer is testing:** Distributed-systems literacy and the ability to reason about consistency trade-offs concretely, not just recite "CAP."

**Ideal Answer:**

**CAP theorem:** In a distributed system you can guarantee at most **two of three** during a network partition:
- **Consistency (C):** every read sees the most recent write (linearizable).
- **Availability (A):** every request gets a (non-error) response.
- **Partition tolerance (P):** the system keeps working despite network splits between nodes.

**In practice, P is unavoidable** (networks *will* partition), so the real choice during a partition is **CP vs AP**:
- **CP:** during a partition, refuse some requests (return errors) to preserve consistency.
- **AP:** during a partition, stay available but possibly return stale data.

**Classifications:**
- **PostgreSQL (with synchronous replication): CP.** It prioritizes consistency; during a partition/failover it may **reject writes** (or block) rather than serve inconsistent data. A single-primary system chooses correctness over availability.
- **Cassandra: AP.** Always accepts writes and may serve **stale reads** during a partition. But it's **tunably consistent**: per-query consistency levels (`ONE`, `QUORUM`, `ALL`); `R + W > N` gives strong consistency at the cost of availability/latency.
- **DynamoDB: AP by default** (eventually consistent reads), but offers **strongly consistent reads** as an option (higher latency and cost), and global tables are multi-master eventually consistent.
- **Redis Cluster: AP-leaning.** It favors availability; during a partition a minority side stops serving, and because replication is async, a failover can **lose writes** that hadn't propagated — so it is not strongly consistent.

**PACELC extension:** CAP only describes behavior *during a partition*. PACELC says: **if Partition, choose A or C; Else (normal operation), choose Latency or Consistency.**
- PostgreSQL (sync): **PC/EC** — consistent during partition, and trades latency for consistency normally.
- Cassandra/DynamoDB: **PA/EL** — available during partition, and favor low latency over strong consistency normally.

**Follow-up questions the interviewer will ask:**

1. *"How does Cassandra give strong consistency if it's AP?"* — Tunable quorums: with `N` replicas, if write consistency `W` + read consistency `R` satisfy `R + W > N` (e.g., `QUORUM` reads and writes), reads always see the latest write. You trade availability/latency for that guarantee per query.

2. *"Is a single-node PostgreSQL CP or AP?"* — CAP only applies to distributed systems. A single node has no partition; it's simply consistent and available until it dies (then unavailable). CAP becomes relevant once you add replicas/failover.

3. *"Why is PACELC more useful than CAP?"* — Partitions are rare; most of the time the system isn't partitioned, and the real everyday trade-off is latency vs consistency (e.g., reading from a lagging replica for speed). PACELC captures that; CAP ignores it.

**Common mistakes that get you rejected:**
- Saying a system "gives up P" — you can't, in a real distributed system; you choose C or A under partition.
- Calling Cassandra strictly AP without mentioning tunable consistency.
- Not knowing PACELC, which most distributed-systems interviewers now expect.

---

### Q56: Cassandra Data Modeling — Partition Keys, Clustering Keys, and Query-First Design
**Company:** Flipkart, Hotstar, Netflix-India  **Difficulty:** 🔴  **Frequency:** 🔥  **Round:** Onsite

**Question:** Explain Cassandra's primary key structure, why secondary indexes are dangerous, and how query-first modeling works. Show a time-series table.

**What interviewer is testing:** Whether you can model for a distributed, no-JOIN, write-optimized store instead of forcing relational habits onto it.

**Ideal Answer:**

**Primary key = (partition key, clustering key(s)):**
- **Partition key** decides **which node** stores the data (hash of partition key → token → node). All rows with the same partition key live together on the same node(s).
- **Clustering key(s)** decide the **sort order of rows *within* a partition**.

**Time-series events table (CQL):**
```sql
CREATE TABLE events_by_device (
    device_id   text,
    event_time  timestamp,
    event_type  text,
    payload     text,
    PRIMARY KEY ((device_id), event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);
```
- `(device_id)` is the partition key → all events for a device are co-located.
- `event_time` is the clustering key → events sorted by time within the partition.
- Query that performs well (hits one partition, uses sort order):
  ```sql
  SELECT * FROM events_by_device
  WHERE device_id = 'sensor-42'
    AND event_time >= '2026-06-01' AND event_time < '2026-06-02';
  ```

**Why secondary indexes are dangerous:**
- A Cassandra secondary index is stored **locally on each node** for that node's data.
- Querying by a secondary-indexed column that is **not** the partition key forces a **scatter-gather across all nodes** (the coordinator must ask every node) → slow, unscalable.
- Especially bad for **high-cardinality** columns (e.g., email) and very **low-cardinality** columns (e.g., boolean). **Avoid secondary indexes for those;** instead create a **separate query-specific table**.

**Query-first design:**
- You **design tables around the queries** you must serve, not around entities. Each access pattern often gets its **own denormalized table** (data is duplicated across tables — that's expected and cheap on writes).
  ```sql
  -- To also query events by type, make a dedicated table:
  CREATE TABLE events_by_type (
      event_type  text,
      event_time  timestamp,
      device_id   text,
      payload     text,
      PRIMARY KEY ((event_type), event_time)
  );
  ```

**Partition sizing limits:**
- Keep a partition **under ~100MB** and avoid **wide rows > ~100K cells** — huge partitions cause memory pressure, slow reads, and compaction problems.
- For high-volume time-series, **bucket** the partition key (e.g., `((device_id, day))`) so each partition stays bounded:
  ```sql
  PRIMARY KEY ((device_id, day_bucket), event_time)
  ```

**Follow-up questions the interviewer will ask:**

1. *"Why duplicate data across multiple tables?"* — Writes are cheap (LSM, sequential), JOINs are impossible, and disk is cheap. Trading storage and write-amplification for fast single-partition reads is the Cassandra way.

2. *"What happens if a partition key has too few distinct values?"* — Data piles into a few huge partitions on a few nodes → hotspots and oversized partitions. Add a bucketing component to spread load.

3. *"When is a secondary index acceptable?"* — Low scatter risk: when you already restrict the query to a single partition and just filter within it, or for moderate-cardinality columns at small scale. Otherwise prefer a materialized view or a separate table.

**Common mistakes that get you rejected:**
- Modeling Cassandra like a normalized relational DB and expecting JOINs.
- Using secondary indexes on high-cardinality columns (scatter-gather killer).
- Ignoring partition size limits, creating unbounded wide partitions.

---

### Q57: Redis Data Structures, Persistence, and Eviction
**Company:** Razorpay, PhonePe, Swiggy, any startup  **Difficulty:** 🟡  **Frequency:** 🔥🔥🔥  **Round:** Phone/Onsite

**Question:** Walk through Redis's core data structures with use cases, explain RDB vs AOF persistence, and the eviction policies. Show a leaderboard.

**What interviewer is testing:** Practical caching/in-memory store knowledge — extremely common in startup interviews.

**Ideal Answer:**

**Core data structures and use cases:**
- **String** — counters (`INCR`), session tokens, cached JSON blobs, feature flags.
- **Hash** — an object with field-level get/set: user profile, shopping cart (`HSET cart:42 item1 2`).
- **List** — ordered, push/pop from ends: simple message queue, activity feed (`LPUSH` / `RPOP`).
- **Set** — unique unordered members: unique visitors, tags, set intersection/union (`SADD`, `SINTER`).
- **Sorted Set (ZSet)** — members ranked by score: **leaderboards**, sliding-window rate limiting, delayed/priority job queues (score = timestamp).

**Persistence — RDB vs AOF:**
- **RDB (snapshot):** periodically forks and dumps the whole dataset to a compact binary file.
  - *Pro:* fast restart, compact, low runtime overhead. *Con:* you can **lose the writes since the last snapshot** (e.g., last few minutes) on crash.
- **AOF (Append-Only File):** logs **every write command**; replayed on restart.
  - *Pro:* far more durable (configurable `fsync` — `everysec` is typical → at most ~1s loss). *Con:* larger file, slower restart, needs periodic rewrite/compaction.
- *Best practice:* enable **both** — AOF for durability, RDB for fast restore/backups.

**Eviction policies (when `maxmemory` is hit):**
- **`noeviction`** — reject writes with an error (default; safest for a system-of-record-ish use).
- **`allkeys-lru`** — evict least-recently-used among all keys (general cache).
- **`volatile-lru`** — LRU but only among keys with a TTL set.
- **`allkeys-lfu` / `volatile-lfu`** — least-*frequently*-used (better when some keys are hot long-term).
- Also `allkeys-random`, `volatile-ttl` (evict nearest expiry).

**Leaderboard with a Sorted Set:**
```text
ZADD leaderboard 1500 "alice"          # add/update score
ZADD leaderboard 2300 "bob"
ZINCRBY leaderboard 50 "alice"         # alice scores +50 -> 1550

ZREVRANGE leaderboard 0 9 WITHSCORES   # top 10 (highest first)
ZREVRANK leaderboard "alice"           # alice's 0-based rank from the top
ZSCORE leaderboard "alice"             # alice's current score
ZCOUNT leaderboard 1000 2000           # how many players in a score band
```

**Follow-up questions the interviewer will ask:**

1. *"How would you do sliding-window rate limiting with a ZSet?"* — Store each request timestamp as a member with score = timestamp in a per-user ZSet. On each request: `ZREMRANGEBYSCORE` to drop entries older than the window, `ZCARD` to count remaining, allow if under the limit, `ZADD` the new timestamp, and set a TTL on the key.

2. *"RDB or AOF for a payments idempotency cache?"* — AOF with `appendfsync everysec` (or even `always` for critical paths) to minimize data loss; ideally AOF + RDB. Pure RDB risks losing recent idempotency keys on crash.

3. *"Why LFU over LRU sometimes?"* — LRU evicts a hot key that simply wasn't touched in the last instant; LFU keeps frequently-accessed keys even after a brief idle period, which better matches skewed access patterns.

**Common mistakes that get you rejected:**
- Confusing List (ordered, dup-allowed) with Set (unique, unordered).
- Saying RDB is fully durable — it can lose writes since the last snapshot.
- Not knowing `noeviction` is the default and can cause write errors when memory fills.

---

### Q58: Zero-Downtime Database Migrations with Expand-Contract
**Company:** Razorpay, PayPal, any fintech  **Difficulty:** 🔴  **Frequency:** 🔥🔥  **Round:** Onsite

**Question:** How do you change a schema on a large, live, high-traffic table without downtime? Explain expand-contract and compare Flyway vs Liquibase.

**What interviewer is testing:** Production safety. Knowing that naive DDL locks tables and takes down services is a senior signal.

**Ideal Answer:**

**The danger:**
- `ALTER TABLE ADD COLUMN ... NOT NULL` **without a default** on a large table rewrites/validates every row under a lock → blocks all reads/writes → outage. (Modern PostgreSQL can add a column with a *constant* default cheaply, but adding `NOT NULL` and backfilling on a huge live table still needs care.)
- Creating an index normally takes a lock that blocks writes.

**Expand-Contract pattern (the safe, multi-deploy approach):**
1. **Expand** — add the new column **nullable** (instant, no rewrite):
   ```sql
   ALTER TABLE orders ADD COLUMN status_v2 text;  -- nullable, cheap
   ```
2. **Backfill in batches** — populate existing rows incrementally to avoid a giant locking transaction:
   ```sql
   UPDATE orders SET status_v2 = status
   WHERE status_v2 IS NULL AND id BETWEEN :lo AND :hi;  -- loop over id ranges
   ```
3. **Add the constraint** — once backfilled, add `NOT NULL` (in PG, validate with low impact):
   ```sql
   ALTER TABLE orders ADD CONSTRAINT orders_status_v2_nn
     CHECK (status_v2 IS NOT NULL) NOT VALID;   -- fast, no full scan
   ALTER TABLE orders VALIDATE CONSTRAINT orders_status_v2_nn;  -- scans without blocking writes
   ```
4. **Deploy new code** that reads/writes `status_v2` (dual-write during transition so both old and new code versions work).
5. **Contract** — once all code uses the new column, drop the old one:
   ```sql
   ALTER TABLE orders DROP COLUMN status;
   ```

**Indexes without downtime:**
```sql
CREATE INDEX CONCURRENTLY idx_orders_status_v2 ON orders (status_v2);
-- No exclusive lock on writes; slower to build, can't run inside a transaction block.
```

**Migration tools — Flyway vs Liquibase:**
- **Flyway:** plain **SQL files** with sequential versioning (`V1__init.sql`, `V2__add_col.sql`). Simple, SQL-native, great when the team thinks in SQL. Rollback is "forward-fix" in the free version. (This very project uses Flyway — `V1__init.sql` … `V6__add_resume_support.sql`.)
- **Liquibase:** **changesets** in XML/YAML/JSON (or SQL). Database-agnostic abstractions and **built-in rollback** support. More structure/ceremony, better for multi-DB portability and explicit rollbacks.

**Online table rebuild:** use **`pg_repack`** to remove bloat / rewrite a table without the long exclusive lock that `VACUUM FULL` takes.

**Follow-up questions the interviewer will ask:**

1. *"Why backfill in batches instead of one UPDATE?"* — A single `UPDATE` over millions of rows is one huge transaction: long lock contention, massive WAL, replication lag spikes, and bloat. Batching keeps each transaction small and lets autovacuum keep up.

2. *"Why CREATE INDEX CONCURRENTLY and what's the catch?"* — It avoids the write-blocking lock, but it's slower, can't run inside a transaction, and can leave an **invalid index** if it fails (you must drop and retry).

3. *"How do you rename a column with zero downtime?"* — You don't rename directly. Add a new column, dual-write, backfill, switch reads, then drop the old — the same expand-contract flow.

**Common mistakes that get you rejected:**
- Running `ALTER TABLE ... ADD COLUMN NOT NULL` / a giant `UPDATE` on a live table and locking it.
- Using `CREATE INDEX` (not `CONCURRENTLY`) on a production table.
- Trying to do the schema + code change in one deploy instead of expand → migrate code → contract.

---

### Q59: Design an Indexing Strategy for a Slow Query
**Company:** Amazon, Uber, Zomato  **Difficulty:** 🔴  **Frequency:** 🔥🔥🔥  **Round:** Onsite

**Question:** Given a slow query, walk through choosing the right index — selectivity, composite order, partial index, expression index, covering index — with before/after `EXPLAIN ANALYZE`.

**What interviewer is testing:** Applied indexing judgment, the bread-and-butter of backend performance work.

**Ideal Answer:**

**The slow query:**
```sql
SELECT order_id, total, created_at
FROM orders
WHERE status = 'active'
  AND LOWER(email) = 'alice@example.com'
ORDER BY created_at DESC
LIMIT 20;
```

**Step 1 — `EXPLAIN ANALYZE` (before):**
```
Limit (actual time=640.2..640.3 rows=20)
  ->  Sort (actual time=640.2..640.2 rows=20)
        ->  Seq Scan on orders (actual time=0.2..620.1 rows=240 loops=1)
              Filter: ((status='active') AND (lower(email)='alice@example.com'))
              Rows Removed by Filter: 4999760
```
Diagnosis: full scan of 5M rows; `LOWER(email)` blocks any plain `email` index; no index supports the sort.

**Step 2 — Check cardinality/selectivity:**
- `email` is **high selectivity** (few rows per value) → excellent index candidate.
- `status='active'` might be **low selectivity** if most orders are active → poor on its own, but useful in a **partial index** if 'active' is the only status queried.

**Step 3 — Build the right index.**

*Expression index* (enables case-insensitive equality):
```sql
CREATE INDEX idx_orders_lower_email ON orders (LOWER(email));
```
*Composite + partial + sorted*, tailored to this exact query (equality column first, then the sort column, filtered to active rows so the index is small):
```sql
CREATE INDEX idx_orders_active_email_created
  ON orders (LOWER(email), created_at DESC)
  WHERE status = 'active';
```
*Covering index* (add `INCLUDE` so the query is satisfied from the index alone — no heap fetch):
```sql
CREATE INDEX idx_orders_active_covering
  ON orders (LOWER(email), created_at DESC)
  INCLUDE (order_id, total)
  WHERE status = 'active';
```

**Step 4 — `EXPLAIN ANALYZE` (after):**
```
Limit (actual time=0.04..0.06 rows=20)
  ->  Index Only Scan using idx_orders_active_covering on orders (actual time=0.03..0.05 rows=20)
        Index Cond: (lower(email) = 'alice@example.com')
        Heap Fetches: 0
```
640ms → <0.1ms. The Index **Only** Scan reads everything from the index (covering + visibility map), the partial `WHERE status='active'` keeps the index tiny, and `created_at DESC` ordering removes the Sort node.

**Key principles recap:**
- **Selectivity:** index high-selectivity columns; low-selectivity columns belong in a **partial index** predicate or later in a composite.
- **Composite order:** equality columns first, then range/sort columns (Q45).
- **Partial index:** `WHERE` clause makes the index smaller and faster for a known filtered subset.
- **Expression index:** matches functional predicates like `LOWER(email)`.
- **Covering index (`INCLUDE`):** avoids heap lookups → Index Only Scan.

**Follow-up questions the interviewer will ask:**

1. *"What's the cost of all these indexes?"* — Every index slows writes (must be maintained on insert/update/delete) and uses disk. Add only indexes that serve real, frequent queries; drop unused ones (check `pg_stat_user_indexes`).

2. *"When does an Index Only Scan still hit the heap?"* — When pages aren't marked all-visible in the visibility map (recent writes, no recent VACUUM) → `Heap Fetches > 0`. Keep VACUUM healthy.

3. *"Why a partial index here instead of just composite?"* — If you only ever query `status='active'`, the partial index excludes all inactive rows, making it dramatically smaller and faster, and it won't be bloated by statuses you never query.

**Common mistakes that get you rejected:**
- Indexing `status` (low selectivity) alone and expecting speedup.
- Forgetting `LOWER(email)` needs an expression index.
- Not knowing covering (`INCLUDE`) indexes enable heap-free Index Only Scans.

---

### Q60: Distributed Transactions — 2PC vs Saga
**Company:** Amazon, Razorpay, PhonePe, Flipkart  **Difficulty:** ⚫  **Frequency:** 🔥🔥  **Round:** Onsite/System Design

**Question:** Explain Two-Phase Commit and the Saga pattern in depth. Cover compensating transactions, choreography vs orchestration, and when to use each.

**What interviewer is testing:** Microservices data-consistency design at staff level — atomicity across services that don't share a database.

**Ideal Answer:**

**The problem:** A single business action spans multiple databases/services (place order → charge payment → reserve inventory). There's no single DB transaction to wrap them, yet you need consistency.

**Two-Phase Commit (2PC):**
- A **coordinator** drives all participants through two phases:
  - **Phase 1 — Prepare:** coordinator asks every participant "can you commit?" Each does the work tentatively, persists it durably, locks resources, and **votes YES/NO**.
  - **Phase 2 — Commit/Abort:** if **all** voted YES, coordinator broadcasts **COMMIT**; if **any** voted NO (or timed out), it broadcasts **ABORT**. Participants finalize accordingly.
- **Guarantees:** atomic commit across participants (strong consistency).
- **Problems:**
  - **Coordinator is a single point of failure** — if it dies after Prepare, participants are **blocked** holding locks.
  - **Blocking protocol** — participants hold locks during the whole round-trip; a network partition can freeze them.
  - **Poor fit for microservices** — long-held locks across services kill availability and throughput.

**Saga pattern:**
- A saga is a **sequence of local transactions**. Each service commits its own local transaction and then **publishes an event/triggers** the next step. No global lock, no coordinator holding everyone.
- **Compensating transactions:** if step N fails, you run **compensations** for the already-completed steps N-1, N-2, … to semantically undo them (you can't "rollback" a committed local txn, so you issue a counteracting one — refund, release reservation).

**Two ways to coordinate a saga:**
- **Choreography (decentralized):** each service **listens for events** and reacts, emitting its own events. No central brain.
  - *Pro:* loosely coupled, no SPOF. *Con:* hard to follow the overall flow, cyclic event dependencies, harder to debug/observe.
- **Orchestration (centralized):** a **saga orchestrator** explicitly tells each service what to do and tracks state.
  - *Pro:* clear, observable flow; easy to add steps and handle failures in one place. *Con:* the orchestrator is a (manageable) central component to maintain.

**Payment saga example (orchestration), happy path then failure:**
```text
Order Service     ->  CREATE order (pending)
Payment Service   ->  CHARGE card
Inventory Service ->  RESERVE stock
Order Service     ->  MARK order confirmed

If RESERVE stock FAILS:
  Compensate Payment   ->  REFUND card        (undo step 2)
  Compensate Order     ->  CANCEL order        (undo step 1)
  => system ends in a consistent "order cancelled, money refunded" state
```
```text
Forward:        Order(create) -> Payment(charge) -> Inventory(reserve) -> Order(confirm)
Compensations:  Order(cancel) <- Payment(refund) <- Inventory(release)  <- (run in reverse on failure)
```

**When to use which:**
- **2PC:** within a **single trusted boundary** — same DBMS, or a few tightly-coupled internal services where you can tolerate locking and need true atomicity (e.g., XA transactions inside one company datacenter). Avoid across the open internet / many microservices.
- **Saga:** **microservices** with independent databases, where availability and loose coupling matter more than instantaneous global consistency. You accept **eventual consistency** and design **idempotent** steps and compensations.

**Follow-up questions the interviewer will ask:**

1. *"Sagas give eventual consistency — how do you handle the in-between inconsistent state?"* — Use a **pending/processing** status visible to users ("payment processing"), make steps **idempotent** (safe to retry), and ensure compensations are reliable. The system is briefly inconsistent but converges.

2. *"What if a compensation itself fails?"* — Retry with backoff (compensations must be idempotent and eventually succeed); persist saga state durably so it can resume; escalate to a dead-letter queue / manual intervention as a last resort. Sagas assume compensations *will* eventually succeed.

3. *"Why is 2PC called a blocking protocol?"* — Between Prepare and Commit, participants must hold their locks and wait for the coordinator's decision. If the coordinator crashes or the network partitions, they can't unilaterally decide and stay blocked, hurting availability.

**Common mistakes that get you rejected:**
- Proposing 2PC across many microservices (blocking + SPOF makes it unsuitable).
- Thinking a saga "rolls back" — it runs **compensating** transactions; committed work can't be undone, only counteracted.
- Forgetting idempotency, which is essential because saga steps and compensations get retried.

---

## Pattern Recap Cheat Sheet

| Pattern | SQL Technique | Key Gotcha | Example Use Case |
|---|---|---|---|
| Nth highest | `DENSE_RANK() OVER (ORDER BY x DESC)` then filter `= N` | `ROW_NUMBER` mishandles ties; `OFFSET` skips duplicates | 3rd highest salary |
| Top-N per group | `ROW_NUMBER() OVER (PARTITION BY g ORDER BY x DESC) <= N` | Filter the window result in an outer query, not in `WHERE` | Top 3 products per category |
| Running total | `SUM(x) OVER (ORDER BY t ROWS UNBOUNDED PRECEDING)` | Default frame is `RANGE` — ties accumulate together unexpectedly | Cumulative revenue over time |
| Moving average | `AVG(x) OVER (ORDER BY t ROWS BETWEEN 6 PRECEDING AND CURRENT ROW)` | Window is rows not days — gaps in dates skew it | 7-day rolling sales average |
| YoY growth | `LAG(x, 12) OVER (ORDER BY month)` | Missing months break the 12-offset; pre-fill the calendar | Year-over-year revenue change |
| Gap-and-island | `t - ROW_NUMBER()*interval` groups consecutive runs | Must order correctly; type of gap (date vs int) changes the trick | Detect contiguous date ranges |
| Consecutive streak | Gap-and-island on a sequence, then `COUNT(*)` per island | One missing row resets the streak — confirm the definition | Longest daily-login streak |
| Mutual friends | Self-`JOIN` on friendship `WHERE a.fid=b.uid AND a.uid<b.fid` | Forgetting symmetry / counting pairs twice | "People you may know" |
| Session ID | Flag new session when `gap > threshold` via `LAG`, then `SUM()` cumulative flag | Per-user partition needed; gap boundary off-by-one | Web analytics sessionization |
| Recursive org chart | `WITH RECURSIVE cte AS (anchor UNION ALL recursive)` | Add a depth/cycle guard to avoid infinite loops | Employee → manager hierarchy |
| Duplicate delete | `DELETE ... USING` keeping `MIN(ctid)`/`MIN(id)` per dup group | Forgetting the keep-one condition deletes all copies | Dedup rows with same email |
| Anti-join | `LEFT JOIN ... WHERE r.id IS NULL` or `NOT EXISTS` | `NOT IN` with a NULL in the subquery returns no rows | Customers with no orders |
| Pivot | `SUM(CASE WHEN col=v THEN x END)` per value, or `crosstab()` | Values must be known up front; dynamic pivot needs PL/pgSQL | Monthly sales columns from rows |
| Median | `PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY x)` | `AVG` is the mean, not the median; ordered-set aggregate needed | Median order value |
| Upsert | `INSERT ... ON CONFLICT (key) DO UPDATE SET ...` | Needs a unique/exclusion constraint on the conflict target | Idempotent record insert |
| Lateral join | `CROSS/LEFT JOIN LATERAL (SELECT ... WHERE l.id=...) sub` | Correlated subquery per row — index the join column | Top 3 orders for each customer |
| MVCC | Rely on snapshot isolation; `xmin`/`xmax` tuple versions | Dead tuples bloat tables — keep VACUUM healthy | Readers don't block writers |
| Isolation levels | `BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE` | Must retry on `40001`; PG maps READ UNCOMMITTED → READ COMMITTED | Prevent write-skew in scheduling |
| Normalization | Split tables to 3NF/BCNF by functional dependencies | Over-normalizing hot read paths costs JOINs — denormalize selectively | Remove update anomalies |
| Sharding | Shard key = common query predicate; consistent hashing | `hash % N` makes resharding move ~all keys; sequential keys hotspot | Scale orders by `customer_id` |
| Partial index | `CREATE INDEX ... WHERE status='active'` | Query predicate must match the index predicate to be used | Index only active rows |
| Covering index | `CREATE INDEX ... INCLUDE (cols)` for Index Only Scan | Heap fetches return if pages aren't all-visible (VACUUM) | Heap-free hot-path lookups |

Created 10-database-sql-interview.md — 60 questions covered.
