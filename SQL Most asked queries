# SQL Interview Questions — Infosys L2/L3 (SQL Only, No DBMS Theory)

Infosys L2/L3 rounds rarely ask 5–6 hard problems. They usually ask **2–4 medium
questions**, then push with:
- "Can you solve it another way?"
- "Can you do it without window functions?"
- "Can you use a JOIN instead of a subquery?"
- "Can you optimize it?"

So the goal isn't just solving each question — it's having **2–3 approaches ready**
for the starred ones below.

Sample tables used throughout:
```
Employee(id, name, salary, departmentId, managerId, hireDate)
Department(id, name)
Customers(id, name)
Orders(id, customerId, amount, orderDate, createdAt)
Person(id, email)
Logs(id, num)
Salary(employeeId, month, salary)
Transactions(id, country, state, amount, trans_date)
```

---

## Part A — Salary Queries (Rank / Nth / Comparison)

### 1. Second Highest Salary ⭐⭐⭐⭐⭐

**Method 1 — MAX + Subquery (no ORDER BY/LIMIT needed)**
```sql
SELECT MAX(salary) AS SecondHighestSalary
FROM Employee
WHERE salary < (SELECT MAX(salary) FROM Employee);
```

**Method 2 — DENSE_RANK()**
```sql
SELECT salary AS SecondHighestSalary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM Employee
) t
WHERE rnk = 2;
```

**Method 3 — LIMIT + OFFSET**
```sql
SELECT (
    SELECT DISTINCT salary
    FROM Employee
    ORDER BY salary DESC
    LIMIT 1 OFFSET 1
) AS SecondHighestSalary;
```

**Method 4 — Correlated subquery (counts how many distinct salaries beat it)**
```sql
SELECT DISTINCT salary
FROM Employee e1
WHERE 1 = (
    SELECT COUNT(DISTINCT salary)
    FROM Employee e2
    WHERE e2.salary > e1.salary
);
```

> **Tie-breaking trap:** if two people share the highest salary, `DENSE_RANK` still
> gives the correct "true" 2nd distinct value; `ROW_NUMBER` would not, since it
> just numbers rows without collapsing duplicates.

---

### 2. Nth Highest Salary ⭐⭐⭐⭐⭐

**LIMIT/OFFSET**
```sql
SELECT DISTINCT salary
FROM Employee
ORDER BY salary DESC
LIMIT 1 OFFSET 5;   -- 6th highest (OFFSET = N-1)
```

**Window function (most robust — handles duplicate salaries correctly)**
```sql
SELECT salary
FROM (
    SELECT salary, DENSE_RANK() OVER (ORDER BY salary DESC) AS rnk
    FROM Employee
) t
WHERE rnk = 6;
```

**Correlated subquery (generalizes Q1's Method 4)**
```sql
SELECT DISTINCT salary
FROM Employee e1
WHERE 5 = (
    SELECT COUNT(DISTINCT salary)
    FROM Employee e2
    WHERE e2.salary > e1.salary
);
```

---

### 3. Employees Above Average Salary ⭐⭐⭐

**Scalar subquery**
```sql
SELECT *
FROM Employee
WHERE salary > (
    SELECT AVG(salary)
    FROM Employee
);
```

---

### 4. Employees Above Their Department's Average Salary ⭐⭐⭐⭐

**Correlated subquery**
```sql
SELECT *
FROM Employee e
WHERE salary > (
    SELECT AVG(salary)
    FROM Employee
    WHERE departmentId = e.departmentId
);
```

**Window function alternative (if asked "without a correlated subquery")**
```sql
SELECT *
FROM (
    SELECT *, AVG(salary) OVER (PARTITION BY departmentId) AS dept_avg
    FROM Employee
) t
WHERE salary > dept_avg;
```
> Good talking point: the window version computes each department's average once
> per partition, while the correlated subquery re-runs the average calculation
> for every single row — worth mentioning as a performance trade-off.

---

### 5. Highest Salary in Each Department ⭐⭐⭐⭐⭐

**GROUP BY (gives only the max value per department, not the employee)**
```sql
SELECT departmentId, MAX(salary)
FROM Employee
GROUP BY departmentId;
```

**JOIN + GROUP BY (with department name)**
```sql
SELECT d.name, MAX(e.salary)
FROM Employee e
JOIN Department d ON e.departmentId = d.id
GROUP BY d.name;
```

**DENSE_RANK() (gives the actual employee row too)**
```sql
SELECT *
FROM (
    SELECT *, DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) rnk
    FROM Employee
) t
WHERE rnk = 1;
```

**Correlated subquery version, if asked "without window functions"**
```sql
SELECT *
FROM Employee e
WHERE salary = (
    SELECT MAX(salary) FROM Employee e2 WHERE e2.departmentId = e.departmentId
);
```

---

### 6. Second Largest Salary Per Department ⭐⭐⭐⭐

```sql
SELECT *
FROM (
    SELECT *, DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) rnk
    FROM Employee
) t
WHERE rnk = 2;
```
(Same template as Q7 with `rnk = 2` instead of `<= 3` — interviewers like to see
you recognize this is the *same pattern*, not a new problem.)

---

### 7. Top 3 Salaries in Every Department ⭐⭐⭐⭐⭐

```sql
WITH t AS (
    SELECT *, DENSE_RANK() OVER (PARTITION BY departmentId ORDER BY salary DESC) AS rnk
    FROM Employee
)
SELECT * FROM t WHERE rnk <= 3;
```
> Discuss `DENSE_RANK` vs `ROW_NUMBER` here: DENSE_RANK could return more than 3 rows
> per department if there are salary ties at rank 3; ROW_NUMBER always caps at exactly 3.

---

### 8. Employees Earning More Than Their Manager ⭐⭐⭐⭐⭐

**Self Join**
```sql
SELECT e.name AS Employee
FROM Employee e
JOIN Employee m ON e.managerId = m.id
WHERE e.salary > m.salary;
```

**Correlated Subquery**
```sql
SELECT name
FROM Employee e
WHERE salary > (
    SELECT salary FROM Employee WHERE id = e.managerId
);
```

---

## Part B — Duplicates

### 9. Find Duplicate Records ⭐⭐⭐⭐⭐

**Just the duplicate keys**
```sql
SELECT email
FROM Employee
GROUP BY email
HAVING COUNT(*) > 1;
```

**Full duplicate rows — via IN**
```sql
SELECT *
FROM Person
WHERE email IN (
    SELECT email FROM Person GROUP BY email HAVING COUNT(*) > 1
);
```

**Full duplicate rows — via window function (avoids the extra self-scan of the IN version)**
```sql
SELECT *
FROM (
    SELECT *, COUNT(*) OVER (PARTITION BY email) AS cnt
    FROM Person
) t
WHERE cnt > 1;
```

---

### 10. Count Duplicate Records ⭐⭐⭐

```sql
SELECT email, COUNT(*) AS duplicate_count
FROM Employee
GROUP BY email
HAVING COUNT(*) > 1;
```
(Same query shape as Q9's first method — just returning the count alongside the key
instead of only the key.)

---

### 11. Delete Duplicate Records ⭐⭐⭐⭐

**Self-join DELETE (keep the smallest id per email)**
```sql
DELETE p1
FROM Person p1
JOIN Person p2
  ON p1.email = p2.email
 AND p1.id > p2.id;
```

**ROW_NUMBER() version**
```sql
DELETE FROM Employee
WHERE id IN (
    SELECT id FROM (
        SELECT id, ROW_NUMBER() OVER (PARTITION BY email ORDER BY id) rn
        FROM Employee
    ) t
    WHERE rn > 1
);
```

**GROUP BY / MIN version — equivalent logic, no window function or self-join needed**
```sql
DELETE FROM Employee
WHERE id NOT IN (
    SELECT MIN(id) FROM Employee GROUP BY email
);
```
> All three delete the same rows. The self-join version is the classic MySQL-style
> answer; `ROW_NUMBER()` generalizes better if you need to keep more than one row
> per group (e.g., "keep the two most recent"); `GROUP BY + MIN` is the simplest
> to read when you only ever keep exactly one row.

---

## Part C — Grouping & Counting

### 12. Employee Count, Department-wise ⭐⭐⭐

```sql
SELECT departmentId, COUNT(*) AS employee_count
FROM Employee
GROUP BY departmentId;
```

---

### 13. Departments Having More Than 5 Employees ⭐⭐⭐⭐

```sql
SELECT departmentId, COUNT(*) AS employee_count
FROM Employee
GROUP BY departmentId
HAVING COUNT(*) > 5;
```
(Tests whether you instinctively reach for `HAVING` instead of trying to filter
group-level conditions in `WHERE` — a very common slip under interview pressure.)

---

### 14. Customers With More Than 3 Orders ⭐⭐⭐⭐

```sql
SELECT customerId
FROM Orders
GROUP BY customerId
HAVING COUNT(*) > 3;
```

---

## Part D — Joins, NULLs & Missing Data

### 15. Customers Who Never Ordered ⭐⭐⭐⭐

**LEFT JOIN**
```sql
SELECT c.name AS Customers
FROM Customers c
LEFT JOIN Orders o ON c.id = o.customerId
WHERE o.id IS NULL;
```

**NOT EXISTS (safe with NULLs, generally the recommended answer)**
```sql
SELECT c.name AS Customers
FROM Customers c
WHERE NOT EXISTS (
    SELECT 1 FROM Orders o WHERE o.customerId = c.id
);
```

**NOT IN (works, but breaks silently if customerId in Orders can be NULL)**
```sql
SELECT name
FROM Customers
WHERE id NOT IN (SELECT customerId FROM Orders WHERE customerId IS NOT NULL);
```

---

## Part E — Window Functions & Time-Series Patterns

### 16. Latest Record Per Customer/User ⭐⭐⭐⭐

**ROW_NUMBER()**
```sql
SELECT *
FROM (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY customerId ORDER BY orderDate DESC) AS rn
    FROM Orders
) t
WHERE rn = 1;
```

**MAX(date) + JOIN, if asked "without window functions"**
```sql
SELECT o.*
FROM Orders o
JOIN (
    SELECT customerId, MAX(orderDate) AS max_date
    FROM Orders
    GROUP BY customerId
) m ON o.customerId = m.customerId AND o.orderDate = m.max_date;
```
> Trap: the JOIN version can return **two rows** for a customer if they have two
> orders with the exact same date — `ROW_NUMBER()` (with a tiebreaker in `ORDER BY`)
> guarantees exactly one. Worth mentioning unprompted.

---

### 17. Running Total ⭐⭐⭐⭐

**Window function**
```sql
SELECT orderId, orderDate, amount,
       SUM(amount) OVER (ORDER BY orderDate) AS running_total
FROM Orders;
```

**Self-join version, if asked "without window functions"**
```sql
SELECT o1.orderId, SUM(o2.amount) AS running_total
FROM Orders o1
JOIN Orders o2 ON o2.orderDate <= o1.orderDate
GROUP BY o1.orderId
ORDER BY o1.orderDate;
```
(Flag this as O(n²) — good to mention proactively.)

---

### 18. Cumulative Salary Per Employee ⭐⭐⭐⭐

```sql
SELECT employeeId, month,
       SUM(salary) OVER (PARTITION BY employeeId ORDER BY month) AS cumulative_salary
FROM Salary;
```
(Same pattern as Q17's running total, just partitioned per employee.)

---

### 19. Rank Employees by Salary ⭐⭐⭐⭐

```sql
SELECT *, RANK() OVER (ORDER BY salary DESC) AS rnk FROM Employee;
```

Know the difference cold — this gets asked almost every time:

| Function | Ties | Gaps after ties |
|---|---|---|
| `ROW_NUMBER()` | Never ties, arbitrary order among equal values | No gaps, strictly sequential |
| `RANK()` | Ties share the same rank | Skips the next rank(s) |
| `DENSE_RANK()` | Ties share the same rank | No gaps — next rank is +1 |

Example with salaries `[100, 90, 90, 80]`:
`ROW_NUMBER` → 1,2,3,4 | `RANK` → 1,2,2,4 | `DENSE_RANK` → 1,2,2,3

---

### 20. Find Consecutive Values ⭐⭐⭐⭐

**LAG/LEAD version (good for "same value appears in adjacent rows")**
```sql
SELECT DISTINCT num
FROM (
    SELECT num,
           LAG(num) OVER (ORDER BY id) AS prev,
           LEAD(num) OVER (ORDER BY id) AS nxt
    FROM Logs
) t
WHERE num = prev OR num = nxt;
```

**Gaps-and-islands version (for "N consecutive calendar days/numbers", not just adjacent rows)**
```sql
SELECT num, MIN(id) AS start_id, COUNT(*) AS streak
FROM (
    SELECT id, num,
           id - ROW_NUMBER() OVER (PARTITION BY num ORDER BY id) AS grp
    FROM Logs
) t
GROUP BY num, grp
HAVING COUNT(*) >= 3;
```
> Know which variant fits the question — "3 consecutive rows with same value" (gaps-and-islands)
> is a different problem from "any value equal to its neighbor" (LAG/LEAD), and interviewers
> deliberately word it ambiguously to see if you ask a clarifying question.

---

### 21. Percentage of Employees in Each Department ⭐⭐⭐⭐

```sql
SELECT departmentId,
       COUNT(*) * 100.0 / SUM(COUNT(*)) OVER () AS percentage
FROM Employee
GROUP BY departmentId;
```

---

### 22. Monthly Sales Report ⭐⭐⭐⭐

```sql
SELECT MONTH(orderDate) AS month, SUM(amount) AS total_sales
FROM Orders
GROUP BY MONTH(orderDate)
ORDER BY month;
```

---

### 23. Monthly Transactions I (LeetCode-style) ⭐⭐⭐⭐⭐

**Q: For each month and country, find the number of transactions and their total
amount, the number of approved transactions and their total amount.**

```sql
SELECT
    DATE_FORMAT(trans_date, '%Y-%m') AS month,
    country,
    COUNT(*)                                              AS trans_count,
    SUM(CASE WHEN state = 'approved' THEN 1 ELSE 0 END)   AS approved_count,
    SUM(amount)                                           AS trans_total_amount,
    SUM(CASE WHEN state = 'approved' THEN amount ELSE 0 END) AS approved_total_amount
FROM Transactions
GROUP BY month, country;
```

**Alternative — Postgres `FILTER` clause (if asked "any way without CASE WHEN?")**
```sql
SELECT
    TO_CHAR(trans_date, 'YYYY-MM') AS month,
    country,
    COUNT(*) AS trans_count,
    COUNT(*) FILTER (WHERE state = 'approved') AS approved_count,
    SUM(amount) AS trans_total_amount,
    SUM(amount) FILTER (WHERE state = 'approved') AS approved_total_amount
FROM Transactions
GROUP BY month, country;
```
> This is a good one to bring up the **conditional aggregation** pattern by name —
> `SUM(CASE WHEN ... THEN x ELSE 0 END)` is the single most reusable trick in
> these interviews (also used for pivoting, in Q24).

---

### 24. Pivot Rows to Columns ⭐⭐⭐⭐

**Conditional aggregation (works on every RDBMS)**
```sql
SELECT
    SUM(CASE WHEN gender = 'M' THEN 1 ELSE 0 END) AS male,
    SUM(CASE WHEN gender = 'F' THEN 1 ELSE 0 END) AS female
FROM Employee;
```

**Native PIVOT (SQL Server-specific, mention as an alternative)**
```sql
SELECT [M], [F]
FROM (SELECT gender FROM Employee) src
PIVOT (COUNT(gender) FOR gender IN ([M], [F])) p;
```

---

### 25. Median Salary ⭐⭐⭐

**PERCENTILE_CONT (SQL Server/Postgres/Oracle)**
```sql
SELECT DISTINCT
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary) OVER () AS median_salary
FROM Employee;
```

**Manual — portable to MySQL**
```sql
SELECT AVG(salary) AS median_salary
FROM (
    SELECT salary,
           ROW_NUMBER() OVER (ORDER BY salary) AS rn,
           COUNT(*) OVER () AS cnt
    FROM Employee
) t
WHERE rn IN ((cnt + 1) / 2, (cnt + 2) / 2);
```

---

## Part F — Miscellaneous

### 26. Find Missing IDs ⭐⭐⭐

**Anti-join style**
```sql
SELECT (id + 1) AS missing_id
FROM Employee
WHERE (id + 1) NOT IN (SELECT id FROM Employee);
```

**Using a numbers/sequence table (cleaner, catches gaps of more than 1)**
```sql
SELECT n.id
FROM (
    SELECT ROW_NUMBER() OVER (ORDER BY (SELECT NULL)) AS id
    FROM Employee   -- any table with enough rows to cover the range
) n
WHERE n.id BETWEEN (SELECT MIN(id) FROM Employee) AND (SELECT MAX(id) FROM Employee)
  AND n.id NOT IN (SELECT id FROM Employee);
```
(Mention this variant if the interviewer says "what if more than one ID is missing in a row?")

---

### 27. Swap a Column's Values (e.g., gender M/F) ⭐⭐⭐

**CASE expression**
```sql
UPDATE Employee
SET gender = CASE WHEN gender = 'M' THEN 'F' ELSE 'M' END;
```

**Bitwise/char-code trick (asked as "can you do it without CASE/IF")**
```sql
UPDATE Employee
SET gender = CHAR(ASCII('M') + ASCII('F') - ASCII(gender));
```

---

### 28. Multi-Table Join Chains (3+ tables) ⭐⭐⭐⭐⭐

This is a near-guaranteed L3 question — they want to see if you can chain joins
across several tables without getting lost, and whether you understand what changes
when you swap `INNER` for `LEFT` partway through the chain.

Extra tables for this question:
```
Project(id, name, departmentId)
Assignment(employeeId, projectId, hoursWorked)
```

**Q: List each employee's name, their department name, the project they're
assigned to, and hours worked — only for employees who are actually assigned to a project.**

**Method 1 — Straight INNER JOIN chain**
```sql
SELECT e.name AS employee, d.name AS department, p.name AS project, a.hoursWorked
FROM Employee e
JOIN Department d ON e.departmentId = d.id
JOIN Assignment a ON e.id = a.employeeId
JOIN Project p ON a.projectId = p.id;
```

**Q (follow-up): Now also include employees who have NO project assignment.**

**Method 2 — Mixing INNER and LEFT JOIN in the same chain**
```sql
SELECT e.name AS employee, d.name AS department, p.name AS project, a.hoursWorked
FROM Employee e
JOIN Department d ON e.departmentId = d.id      -- every employee has a department: INNER is fine
LEFT JOIN Assignment a ON e.id = a.employeeId    -- assignment may not exist: LEFT
LEFT JOIN Project p ON a.projectId = p.id;       -- must stay LEFT once a.projectId can be NULL
```
> **Trap interviewers set here:** if you write `JOIN Project p` (INNER) after a
> `LEFT JOIN Assignment`, it silently cancels out the LEFT JOIN's effect — any
> employee with no assignment gets filtered out again, because `a.projectId` is
> NULL and can't match `p.id` in an INNER join. Once you go LEFT, every join
> *after* it in the chain touching that nullable key must also be LEFT (or use
> an explicit `ON ... OR a.projectId IS NULL`). This "silent inner join" bug is
> one of the most common things they test at L3.

**Method 3 — Rewritten as correlated subqueries instead of a join chain (if asked "avoid joins")**
```sql
SELECT
    e.name AS employee,
    (SELECT d.name FROM Department d WHERE d.id = e.departmentId) AS department,
    (SELECT p.name FROM Project p
       JOIN Assignment a ON a.projectId = p.id
       WHERE a.employeeId = e.id LIMIT 1) AS project,
    (SELECT a.hoursWorked FROM Assignment a
       WHERE a.employeeId = e.id LIMIT 1) AS hoursWorked
FROM Employee e;
```
(Works, but is slower and messier for 3+ hops — good to mention as a trade-off, not
necessarily to lead with.)

**Q: Total hours worked per department, across all projects.**
```sql
SELECT d.name AS department, SUM(a.hoursWorked) AS total_hours
FROM Department d
JOIN Employee e ON e.departmentId = d.id
JOIN Assignment a ON a.employeeId = e.id
GROUP BY d.name;
```

**Tips for join-chain questions:**
- Always alias every table (`e`, `d`, `a`, `p`) — unaliased multi-joins are hard to read and interviewers dock you for it.
- Decide join order by *cardinality reasoning*, not just table order: start from the table that guarantees a row for what you're counting (e.g., start from `Department` if you want every department even with zero employees).
- State out loud, before writing code, which joins can drop rows (INNER) vs which must preserve them (LEFT/RIGHT) — this is usually what they're actually scoring.
- If asked for "employees in no project AND departments with no employees" simultaneously, that's a `FULL OUTER JOIN` — mention it even if the RDBMS in the question (MySQL) doesn't support it natively, and note the `UNION` of two `LEFT JOIN`s as the MySQL workaround:
```sql
SELECT e.name, d.name
FROM Employee e LEFT JOIN Department d ON e.departmentId = d.id
UNION
SELECT e.name, d.name
FROM Employee e RIGHT JOIN Department d ON e.departmentId = d.id;
```

---

## Cheat Sheet: Which Question Gets Which "Do It Another Way"

| Question | Alternative approaches to have ready |
|---|---|
| Second/Nth highest salary | `MAX()` subquery, `DENSE_RANK()`, `LIMIT/OFFSET`, correlated subquery |
| Employees above dept average | Correlated subquery, window `AVG() OVER (PARTITION BY ...)` |
| Highest salary per department | `GROUP BY`, JOIN + `GROUP BY`, `DENSE_RANK()`, correlated subquery |
| Employees > manager | Self JOIN, correlated subquery |
| Delete duplicates | Self-join `DELETE`, `ROW_NUMBER()`, `GROUP BY` + `MIN(id)` |
| Duplicate records | `GROUP BY`/`HAVING`, `IN` subquery, window `COUNT() OVER()` |
| Customers never ordered | `LEFT JOIN`, `NOT EXISTS`, `NOT IN` |
| Latest record per user | `ROW_NUMBER()`, `MAX(date)` + JOIN |
| Running/cumulative total | Window `SUM() OVER()`, self-join |
| Top N per group | `DENSE_RANK()` vs `ROW_NUMBER()` (know the difference) |
| Consecutive values | `LAG`/`LEAD`, gaps-and-islands |
| Monthly transactions / conditional aggregation | `CASE WHEN` inside `SUM()`, Postgres `FILTER` |
| Multi-table join chains | INNER-only chain, mixed INNER+LEFT chain, correlated subqueries, `FULL OUTER` / `UNION` of two `LEFT JOIN`s |

---

## Must-Know SQL Toolkit for Infosys L3

- `GROUP BY`, `HAVING`
- All joins — especially **self join** and **left join**, and multi-table join chains
- Subqueries: nested, scalar, and **correlated**
- CTEs (`WITH`)
- `CASE WHEN` — especially conditional aggregation (`SUM(CASE WHEN ...)`)
- Window functions: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LAG()`, `LEAD()`, `SUM()/COUNT()/AVG() OVER()`
- `EXISTS` / `NOT EXISTS` vs `IN` / `NOT IN` — and **why NOT IN breaks on NULLs**
- Aggregates: `COUNT`, `SUM`, `AVG`, `MIN`, `MAX`
- `DISTINCT`, `ORDER BY`, `LIMIT`/`OFFSET`

If you can solve everything above and explain **at least two approaches** for every
⭐⭐⭐⭐⭐/⭐⭐⭐⭐ question, you're well prepared for the SQL portion of an Infosys L2/L3
interview. DBMS theory (ACID, normalization, indexing, transactions) is a separate
round of prep — worth doing next.
