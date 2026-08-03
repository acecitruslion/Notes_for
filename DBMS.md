# DBMS + SQL — Complete Interview Notes
### Infosys Specialist Programmer Prep

Structure: Definition → Example → Why it matters. Crisp, exam/interview-ready.

---

## 1. DBMS Basics

- **DBMS**: Software to store, manage, and retrieve data (e.g., MySQL, Oracle).
- **RDBMS**: DBMS that stores data in tables with rows/columns and enforces relationships via keys.
- **DBMS vs RDBMS**: DBMS may not enforce relationships/normalization; RDBMS does, using keys, constraints, and ACID.
- **Table/Relation**: Structured set of rows and columns.
- **Tuple/Row**: A single record.
- **Attribute/Column**: A field/property.
- **Schema vs Instance**: Schema = structure/blueprint (rarely changes). Instance = actual data at a moment in time (changes often).
- **Degree vs Cardinality**: Degree = number of columns. Cardinality = number of rows.
- **View**: Virtual table from a saved query; doesn't store data itself.

**Say in interview:** "RDBMS is a subset of DBMS that stores data in tabular form and enforces relationships and constraints between tables — that's what enables normalization."

---

## 2. Keys ⭐

- **Primary Key** – uniquely identifies each row, no NULLs, one per table.
- **Foreign Key** – column referencing another table's PK; maintains relationships.
- **Candidate Key** – any minimal column set that could qualify as PK.
- **Super Key** – any column set that uniquely identifies a row (may have extra, non-minimal columns).
- **Composite Key** – PK made of 2+ columns together, e.g., `(OrderID, ProductID)`.
- **Unique Key** – ensures uniqueness like PK but allows **one NULL**; multiple allowed per table.

```sql
CREATE TABLE Student (
  RollNo INT PRIMARY KEY,
  Email VARCHAR(50) UNIQUE,
  Aadhar VARCHAR(12) UNIQUE
);
```

**Differences:**
- **PK vs Unique Key** – PK: no NULL, one per table. Unique: one NULL allowed, multiple per table.
- **Candidate Key vs Super Key** – every candidate key is a super key (minimal); not every super key is minimal enough to be a candidate key.
- **PK vs FK** – PK identifies its own row; FK references another table's PK.

---

## 3. Constraints

- **NOT NULL** – column can't be NULL.
- **UNIQUE** – all values distinct (one NULL allowed).
- **PRIMARY KEY** – NOT NULL + UNIQUE.
- **FOREIGN KEY** – enforces valid reference to another table's PK.
- **CHECK** – restricts values by condition, e.g. `CHECK (Age >= 18)`.
- **DEFAULT** – sets a default value if none provided.
- **Referential Integrity** – FK value must match an existing PK or be NULL (no orphan rows).
- **Data Integrity** – overall accuracy/consistency of data; types: entity, referential, domain, user-defined integrity.

---

## 4. Normalization ⭐⭐⭐

**Why?** Reduce redundancy, avoid anomalies.
- **Insert anomaly** – can't insert data without unrelated data also present.
- **Update anomaly** – same data must be updated in multiple places → inconsistency risk.
- **Delete anomaly** – deleting one record accidentally removes other useful data.

**Dependencies:**
- **Functional Dependency (A → B)** – value of A determines value of B. E.g. `RollNo → Name`.
- **Partial Dependency** – non-key column depends on only *part* of a composite key.
- **Transitive Dependency** – non-key column depends on another non-key column, not directly on the key.

### Step-by-step example

**Unnormalized (UNF):**

| StudentID | StudentName | Courses | InstructorPhone |
|---|---|---|---|
| 1 | Ravi | Maths, Science | 9990001111 |
| 2 | Priya | Science | 9990002222 |

Problem: `Courses` isn't atomic — multiple values in one cell.

**1NF — atomic values, no repeating groups**

| StudentID | StudentName | Course | InstructorPhone |
|---|---|---|---|
| 1 | Ravi | Maths | 9990001111 |
| 1 | Ravi | Science | 9990002222 |
| 2 | Priya | Science | 9990002222 |

Still has anomalies: `StudentName` and `InstructorPhone` repeat → insert anomaly (can't add a course with no student), update anomaly (name change → multiple rows), delete anomaly (dropping Maths might lose the fact the course exists).

**2NF — remove partial dependency** (PK = composite `StudentID + Course`; `InstructorPhone` depends only on `Course`, not the full key)

**Enrollment** `(StudentID, Course)` &nbsp;&nbsp; **CourseInstructor** `(Course, InstructorPhone)`

Partial dependency redundancy removed. Transitive dependency issues can still exist elsewhere.

**3NF — remove transitive dependency**

| StudentID | StudentName | DeptID | DeptName |
|---|---|---|---|
| 1 | Ravi | D1 | Computer Science |

`DeptName` depends on `DeptID`, which depends on `StudentID` → transitive. Split:

**Student** `(StudentID, StudentName, DeptID)` &nbsp;&nbsp; **Department** `(DeptID, DeptName)`

Update/insert/delete anomalies around department data are now fixed.

**BCNF — every determinant must be a candidate key** (stricter, handles overlapping candidate key edge cases 3NF misses)

| Student | Subject | Teacher |
|---|---|---|
| Ravi | Maths | Mr. A |
| Ravi | Science | Mr. B |

If each Teacher teaches only one Subject (`Teacher → Subject`) but Teacher isn't a candidate key → violates BCNF even though it's in 3NF. Split into **StudentTeacher** and **TeacherSubject**.

### Quick summary

| Normal Form | Fixes | Still prone to |
|---|---|---|
| 1NF | Non-atomic values, repeating groups | Insert/Update/Delete anomalies |
| 2NF | Partial dependency | Transitive dependency |
| 3NF | Transitive dependency | Rare overlapping-candidate-key anomalies |
| BCNF | Non-candidate-key determinants | Practically anomaly-free |

**One-liner:** "1NF = atomicity, 2NF = no partial dependency, 3NF = no transitive dependency, BCNF = every determinant is a candidate key."

### Denormalization

Intentionally reintroducing redundancy (merging tables / duplicating columns) to speed up reads by reducing JOINs.

**Example:** storing `CustomerName` directly in `Orders` even though it's already in `Customers`, to avoid a JOIN on every read — at the cost of updating it in two places if it changes.

| Aspect | Normalization | Denormalization |
|---|---|---|
| Goal | Reduce redundancy, avoid anomalies | Improve read performance |
| Redundancy | Minimal | Intentionally increased |
| Writes | Faster/safer | Slower/riskier |
| Reads | Slower (more JOINs) | Faster (fewer JOINs) |
| Used in | OLTP | OLAP / reporting |

---

## 5. Transactions ⭐⭐⭐

A **transaction** = sequence of operations treated as one logical unit of work.

```sql
BEGIN TRANSACTION;
UPDATE Account SET Balance = Balance - 500 WHERE ID = 1;
UPDATE Account SET Balance = Balance + 500 WHERE ID = 2;
COMMIT;
```

- **COMMIT** – saves changes permanently.
- **ROLLBACK** – undoes changes since last commit.
- **SAVEPOINT** – checkpoint within a transaction to roll back to partially.
- **States**: Active → Partially Committed → Committed / Failed → Aborted.

### ACID Properties
- **Atomicity** – all or nothing.
- **Consistency** – DB moves between valid states, respecting constraints.
- **Isolation** – concurrent transactions don't interfere.
- **Durability** – committed changes survive crashes.

**Example:** Bank transfer of ₹500, A → B. Atomicity ensures debit+credit both happen or neither; Durability ensures it survives a crash right after commit.

---

## 6. Concurrency Control ⭐⭐

**Problems from uncontrolled concurrency:**
- **Dirty Read** – reading another transaction's uncommitted data (which may later roll back).
- **Non-repeatable Read** – re-reading the same row gives a different value because another transaction updated & committed it in between.
- **Phantom Read** – re-running the same query returns a different row set because another transaction inserted/deleted matching rows.
- **Lost Update** – two transactions update the same data; one overwrites the other.

### Locking
- **Shared Lock (S)** – multiple transactions can hold it for reading; blocks writes.
- **Exclusive Lock (X)** – only one transaction can hold it (for writing); blocks all other locks.

### Two-Phase Locking (2PL)
1. **Growing phase** – acquire locks only, can't release.
2. **Shrinking phase** – release locks only, can't acquire.

Guarantees serializability. *Strict 2PL* releases all locks only at commit/rollback — avoids cascading rollbacks.

### Deadlock
Two+ transactions wait on each other's locks forever.
- **Causes**: mutual exclusion, hold-and-wait, no preemption, circular wait.
- **Prevention**: acquire all locks upfront, enforce lock ordering, wait-die/wound-wait schemes.
- **Detection**: wait-for graph — a cycle means deadlock → abort one transaction (victim selection) or use timeouts.

### Isolation Levels (weakest → strongest)

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| Read Uncommitted | ✅ Possible | ✅ Possible | ✅ Possible |
| Read Committed | ❌ | ✅ Possible | ✅ Possible |
| Repeatable Read | ❌ | ❌ | ✅ Possible |
| Serializable | ❌ | ❌ | ❌ |

Higher isolation = more consistency, less concurrency (performance trade-off).

---

## 7. Indexing ⭐⭐⭐

An **index** is a data structure that speeds up data retrieval (like a book's index) — avoids full table scans.

```
                         INDEXING IN DBMS
                               │
              ┌────────────────┴────────────────┐
              │                                 │
      Primary Index                     Secondary Index
              │                                 │
      ┌───────┴────────┐              ┌─────────┴─────────┐
      │                │              │                   │
   Dense Index     Sparse Index   Clustered Index   Non-Clustered Index
                                           │
                                           │
                                  Multi-Level Index
                                           │
                                  ┌────────┴────────┐
                                  │                 │
                               B-Tree           B+ Tree
                                           │
                                      Hash Index
                                           │
                                      Bitmap Index
```

- **Dense Index** – an index entry for **every** record.
- **Sparse Index** – an index entry for **only some** records (one per block) — less storage, slightly slower lookup.
- **Clustered Index** – physically sorts/stores table data in index order; only **one** per table (usually PK). Like a dictionary — physically in order.
- **Non-Clustered Index** – separate structure with pointers to actual rows; **multiple** allowed per table. Like a book's back index.
- **B+ Tree** – data stored only at leaf nodes, leaves linked together → fast for both exact lookup and **range queries**; what most RDBMS use.

| Clustered | Non-clustered |
|---|---|
| Stores actual data in sorted order | Stores pointers to data |
| Only one per table | Multiple per table |
| Faster range queries | Faster specific/point lookups |

**Trade-off**: faster reads, but slower INSERT/UPDATE/DELETE (index must update too) + extra storage. So index only columns frequently used in WHERE/JOIN/ORDER BY.

---

## 8. Joins ⭐⭐⭐

```sql
-- INNER: only matching rows
SELECT * FROM A INNER JOIN B ON A.id = B.id;

-- LEFT: all of A + matched B (NULL if no match)
SELECT * FROM A LEFT JOIN B ON A.id = B.id;

-- RIGHT: all of B + matched A
SELECT * FROM A RIGHT JOIN B ON A.id = B.id;

-- FULL OUTER: all rows from both, NULL where no match
SELECT * FROM A FULL JOIN B ON A.id = B.id;

-- SELF: table joined with itself (e.g., employee-manager)
SELECT e.Name, m.Name AS Manager FROM Employee e JOIN Employee m ON e.MgrID = m.EmpID;

-- CROSS: cartesian product (every row x every row)
SELECT * FROM A CROSS JOIN B;
```

- **JOIN vs Subquery** – JOIN combines tables directly into one result set; a subquery nests a query inside another, often used for filtering rather than combining columns.
- **Joining 3+ tables** – chain JOIN clauses with proper ON conditions between each pair.

---

## 9. SQL Command Types & Key Comparisons

- **DDL** – CREATE, ALTER, DROP, TRUNCATE (define/modify structure).
- **DML** – INSERT, UPDATE, DELETE (modify data).
- **DQL** – SELECT (retrieve data).
- **DCL** – GRANT, REVOKE (permissions).
- **TCL** – COMMIT, ROLLBACK, SAVEPOINT (transaction control).

### DELETE vs TRUNCATE vs DROP
| | DELETE | TRUNCATE | DROP |
|---|---|---|---|
| Type | DML | DDL | DDL |
| Removes | Rows (WHERE optional) | All rows | Entire table structure |
| Rollback | Yes | Usually no (auto-commit) | No |
| WHERE clause | Yes | No | No |
| Speed | Slower (row-by-row, logged) | Faster | Fastest |

### Other must-know comparisons
- **WHERE vs HAVING** – WHERE filters rows *before* grouping; HAVING filters groups *after* GROUP BY/aggregation.
- **UNION vs UNION ALL** – UNION removes duplicates (slower, sorts internally); UNION ALL keeps duplicates (faster).
- **CHAR vs VARCHAR** – CHAR = fixed length, space-padded, faster for fixed data. VARCHAR = variable length, stores only actual chars, saves space.

```sql
SELECT DeptID, COUNT(*) FROM Employee
GROUP BY DeptID
HAVING COUNT(*) > 5;
```

**Aggregate Functions**: `COUNT()`, `SUM()`, `AVG()`, `MIN()`, `MAX()` — operate on a row set, return a single value.

---

## 10. Database Objects

- **View** – virtual table from a saved query; simplifies complex queries, adds a security layer (restrict columns/rows).
```sql
CREATE VIEW HighEarners AS
SELECT Name, Salary FROM Employee WHERE Salary > 100000;
```
- **Stored Procedure** – precompiled block of SQL, callable by name; reduces network round-trips, reusable logic.
```sql
CREATE PROCEDURE GetEmployee(IN empId INT)
BEGIN
  SELECT * FROM Employee WHERE ID = empId;
END;
CALL GetEmployee(101);
```
- **Function** – like a procedure, but **must return a value** and can be used inside a SQL expression/SELECT.
- **Trigger** – code that auto-executes on an event (INSERT/UPDATE/DELETE) on a table; used for auditing, enforcing business rules.
```sql
CREATE TRIGGER before_insert_salary
BEFORE INSERT ON Employee
FOR EACH ROW
SET NEW.CreatedAt = NOW();
```
- **Cursor** – processes a query's result set **row-by-row** (loop), typically inside a stored procedure.

**Differences:**
- **Procedure vs Function** – Function must return a value, usable in SELECT; Procedure may not return a value, called separately.
- **View vs Table** – Table physically stores data; View is a saved query, shows data dynamically without storing it.
- **Stored Procedure vs Trigger** – SP is explicitly called; Trigger auto-fires on a DB event.

---

## 11. ER Model

- **Entity** – real-world object with attributes (e.g., Student, Order).
- **Attribute** – property of an entity (e.g., name, price).
- **Relationship** – association between entities (e.g., Student *enrolls in* Course).
- **Strong Entity** – has its own PK, exists independently.
- **Weak Entity** – no PK of its own; depends on a strong entity via FK + partial key (e.g., `Dependent` of an `Employee`).

**ER Diagram notation:** Entity → rectangle, Attribute → oval, Relationship → diamond, Key attribute → underlined oval.

**Relationship types:**
- **One-to-One** – e.g., Person ↔ Passport.
- **One-to-Many** – e.g., Customer → many Orders.
- **Many-to-Many** – e.g., Students ↔ Courses.

---

## 12. Storage & Internals (Good to Know)

- **B Tree vs B+ Tree** – B Tree stores data in internal + leaf nodes, no linked leaves (slower range queries). B+ Tree stores data **only** at leaf nodes, leaves linked → faster range queries; used by most RDBMS (e.g., MySQL InnoDB).
- **Hash Index** – maps key → bucket via hash function; very fast for equality (`=`) lookups, but **can't do range queries** (`<`, `>`, `BETWEEN`).
- **Pages and Blocks** – Block = smallest disk I/O unit (e.g., 4KB/8KB). Page = DB's logical storage unit, usually maps to one or more blocks; rows live inside pages.
- **Query Optimization** – the query optimizer picks the cheapest execution plan (which index, join order, join algorithm) using table/index statistics, to minimize I/O + CPU cost.
- **Execution Plan** – step-by-step roadmap the engine follows to run a query (index scan vs full table scan, join method); viewed via `EXPLAIN` in MySQL/PostgreSQL — helps spot slow queries/missing indexes.

---

## 13. SQL — Practice Fluency ⭐⭐⭐

**Core syntax:** SELECT/WHERE, ORDER BY, GROUP BY, HAVING, aggregates, DISTINCT, CASE WHEN, all JOIN types.

- **Subquery vs Correlated Subquery** – correlated subquery depends on the outer query's current row and runs once per outer row (repeatedly); normal subquery runs once independently.
- **CTE** – `WITH temp AS (...) SELECT ...` — makes complex/nested queries readable.
- **Window Functions** – `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()` — assign ranking/numbering without collapsing rows into groups.
- **PARTITION BY** – resets the window function's calculation per group (like a GROUP BY that doesn't collapse rows).

**Common interview query patterns to practice:**
- Nth highest salary → `DENSE_RANK()` or subquery with `LIMIT`/`OFFSET`.
- Top N per group → `ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)`.
- Employees earning more than their manager → self join.
- Find/delete duplicate rows → `GROUP BY ... HAVING COUNT(*) > 1`, or `ROW_NUMBER()`.
- Department-wise counts / highest salary → `GROUP BY` + aggregate.
- Customers with no orders → `LEFT JOIN` + `WHERE right.col IS NULL`.
- Latest record per customer → window function or correlated subquery.
- Running total → `SUM(...) OVER (ORDER BY date)`.
- Consecutive-day/activity problems → window function comparing a row to the previous row.

---

## 14. Interview Favorite Comparisons — Quick Recap

| Comparison | Key Difference |
|---|---|
| DBMS vs RDBMS | RDBMS enforces tabular structure + relationships + ACID |
| Primary Key vs Unique Key | PK: no NULL, 1 per table; Unique: 1 NULL allowed, multiple per table |
| Primary Key vs Foreign Key | PK identifies own row; FK references another table's PK |
| Candidate Key vs Super Key | Every candidate key is a minimal super key; not every super key is minimal |
| DELETE vs TRUNCATE vs DROP | DELETE = rows (DML, rollback); TRUNCATE = all rows (DDL, fast); DROP = whole table |
| CHAR vs VARCHAR | CHAR = fixed length; VARCHAR = variable length, space-efficient |
| WHERE vs HAVING | WHERE filters before grouping; HAVING filters after grouping (aggregates) |
| UNION vs UNION ALL | UNION removes duplicates; UNION ALL keeps them (faster) |
| Clustered vs Non-clustered Index | Clustered = data physically sorted; Non-clustered = separate pointer structure |
| Dense vs Sparse Index | Dense = entry per record; Sparse = entry per block |
| B Tree vs B+ Tree | B+ Tree stores data only at leaves + linked leaves (range-query friendly) |
| 3NF vs BCNF | BCNF is stricter: every determinant must be a candidate key |
| View vs Table | View = virtual, query-based, no storage; Table = physically stores data |
| Stored Procedure vs Trigger | SP = explicitly called; Trigger = auto-fires on a DB event |
| Procedure vs Function | Function must return a value, usable in SELECT; Procedure need not |
| Normalization vs Denormalization | Normalization favors integrity/writes; Denormalization favors read speed |
| Subquery vs Correlated Subquery | Correlated subquery re-runs per outer row; normal subquery runs once |

---

## 15. Priority Order (if short on time)

**Tier 1 (must be strong):** Normalization → Keys → ACID/Transactions → Joins + SQL → Indexing

**Tier 2 (good understanding):** Concurrency + Isolation Levels → Constraints → ER Model

**Tier 3 (quick definitions only):** DBMS/RDBMS basics → Schema/Instance/Degree/Cardinality → Views/Triggers/Cursors/Procedures/Functions → SQL command types → Storage internals (B-Tree vs B+Tree, Hash Index, Pages/Blocks, Query Optimization, Execution Plan)

**Skip entirely (very unlikely to be asked in depth):** Relational algebra/calculus, serializability proofs, precedence graphs, timestamp protocols, recovery/log-based algorithms, RAID/file organization internals, distributed DBs.

---

### Quick Interview Tip
For any topic: **Definition → 1-line example → why it matters.** Keeps answers crisp and confident without rambling.

*Once this is solid, shift remaining prep time to: projects, SQL coding practice, OOP, OS, DSA, system design.*
