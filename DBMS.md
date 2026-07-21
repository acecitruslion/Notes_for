# DBMS + SQL — Interview Notes (Infosys L2/L3)

Simple, exam-ready notes. Each topic = short definition + example. Use this alongside the checklist.

---

## 1. DBMS Basics

- **DBMS**: Software to store, manage, and retrieve data (e.g., MySQL, Oracle).
- **RDBMS**: DBMS that stores data in tables (relations) with rows/columns and enforces relationships via keys.
- **DBMS vs RDBMS**: DBMS may not enforce relationships or normalization; RDBMS does, using keys and constraints.
- **Table/Relation**: A structured set of rows and columns.
- **Tuple/Row**: A single record in a table.
- **Attribute/Column**: A field/property of a table.
- **Schema**: The structure/blueprint of the database (tables, columns, types) — doesn't change often.
- **Instance**: The actual data in the DB at a given moment — changes frequently.
- **Degree**: Number of columns (attributes) in a table.
- **Cardinality**: Number of rows (tuples) in a table.
- **Entity**: A real-world object represented in the DB (e.g., Student, Order).
- **Strong Entity**: Has its own primary key, exists independently.
- **Weak Entity**: No primary key of its own; depends on a strong entity (uses a foreign key + partial key).
- **View**: A virtual table based on a SQL query; doesn't store data physically.

---

## 2. Keys ⭐

- **Primary Key**: Uniquely identifies each row; can't be NULL, only one per table.
- **Foreign Key**: Column referencing the Primary Key of another table; maintains relationships.
- **Candidate Key**: Any column(s) that could qualify as Primary Key (minimal, unique).
- **Super Key**: Any set of columns that uniquely identifies a row (can have extra columns).
- **Composite Key**: Primary key made of 2+ columns together.
- **Unique Key**: Ensures uniqueness but allows one NULL; a table can have multiple.

**Differences:**
- **PK vs Unique Key**: PK doesn't allow NULL, only one per table; Unique Key allows one NULL, multiple allowed.
- **Candidate Key vs Super Key**: Every candidate key is a super key, but not every super key is minimal enough to be a candidate key.

---

## 3. Constraints

- **NOT NULL**: Column can't have NULL values.
- **UNIQUE**: All values in column must be distinct.
- **PRIMARY KEY**: NOT NULL + UNIQUE combined.
- **FOREIGN KEY**: Enforces valid reference to another table's PK.
- **CHECK**: Restricts values based on a condition (e.g., age > 18).
- **DEFAULT**: Sets a default value if none provided.
- **Referential Integrity**: Ensures FK values always match an existing PK (no orphan records).

---

## 4. Normalization ⭐⭐⭐

**Why?** To reduce data redundancy and avoid anomalies.

- **Data Redundancy**: Same data repeated unnecessarily across rows.
- **Insert Anomaly**: Can't insert data without unrelated data also being present.
- **Update Anomaly**: Same data must be updated in multiple places — risk of inconsistency.
- **Delete Anomaly**: Deleting one record accidentally removes other useful data.

**Dependencies:**
- **Functional Dependency (FD)**: Column B depends on Column A (A → B).
- **Partial Dependency**: Non-key column depends on only part of a composite key.
- **Transitive Dependency**: Non-key column depends on another non-key column, not directly on the key.

### Normal Forms — Detailed

**Starting point — Unnormalized Table (UNF):**

| StudentID | StudentName | Courses            | InstructorPhone |
|-----------|-------------|---------------------|------------------|
| 1         | Ravi        | Maths, Science       | 9990001111       |
| 2         | Priya       | Science               | 9990002222       |

Problem: the `Courses` column holds multiple values in one cell (not atomic). This table can suffer **all anomalies** — insert, update, and delete — because everything is jammed into one place.

---

**1NF — First Normal Form**
- **Rule**: Every column must have atomic (single) values — no multi-valued or repeating columns.
- **Fix**: Split multi-valued rows into separate rows.

| StudentID | StudentName | Course  | InstructorPhone |
|-----------|-------------|---------|------------------|
| 1         | Ravi        | Maths   | 9990001111       |
| 1         | Ravi        | Science | 9990002222       |
| 2         | Priya       | Science | 9990002222       |

- **Anomaly still present**: Data is now atomic, but `StudentName` repeats for every course (redundancy), and `InstructorPhone` repeats too.
  - **Insert anomaly**: Can't add a new course unless a student is enrolled in it.
  - **Update anomaly**: If Ravi's name changes, you must update it in multiple rows.
  - **Delete anomaly**: If Ravi drops Maths (delete that row), you might lose the fact that Maths course exists at all (if no other student takes it).

---

**2NF — Second Normal Form**
- **Rule**: Must be in 1NF + **no partial dependency** — i.e., no non-key column should depend on only *part* of a composite primary key. (Only relevant when the PK is composite, e.g., `StudentID + Course`.)
- Here, `InstructorPhone` depends only on `Course` (part of the key), not on the full key `(StudentID, Course)` → partial dependency.
- **Fix**: Split into two tables.

**Enrollment**

| StudentID | Course  |
|-----------|---------|
| 1         | Maths   |
| 1         | Science |
| 2         | Science |

**CourseInstructor**

| Course  | InstructorPhone |
|---------|------------------|
| Maths   | 9990001111       |
| Science | 9990002222       |

- **Anomaly removed**: Partial dependency (and the redundant repetition of `InstructorPhone`) is gone.
- **Anomaly still present**: Transitive dependency issues can still exist in other tables (see 3NF below) — e.g., if we had also kept `StudentName` next to `StudentID` repeatedly, or if one column depends on another non-key column.

---

**3NF — Third Normal Form**
- **Rule**: Must be in 2NF + **no transitive dependency** — a non-key column should not depend on another non-key column (it must depend only on the primary key directly).
- **Example problem table:**

| StudentID | StudentName | DeptID | DeptName   |
|-----------|-------------|--------|------------|
| 1         | Ravi        | D1     | Computer Science |
| 2         | Priya       | D2     | Electronics       |

Here, `DeptName` depends on `DeptID`, and `DeptID` depends on `StudentID` → `DeptName` is transitively dependent on `StudentID` through `DeptID`.

- **Fix**: Split into two tables.

**Student**

| StudentID | StudentName | DeptID |
|-----------|-------------|--------|
| 1         | Ravi        | D1     |
| 2         | Priya       | D2     |

**Department**

| DeptID | DeptName          |
|--------|-------------------|
| D1     | Computer Science  |
| D2     | Electronics       |

- **Anomaly removed**: Transitive dependency gone.
  - **Update anomaly fixed**: Department name is now stored once — no risk of inconsistent updates.
  - **Insert anomaly fixed**: Can add a new department even with no students yet.
  - **Delete anomaly fixed**: Deleting a student doesn't wipe out department info.

---

**BCNF — Boyce-Codd Normal Form**
- **Rule**: Stricter version of 3NF — for every functional dependency `A → B`, `A` must be a **candidate key**. Needed when a table has multiple overlapping candidate keys.
- **Example problem** (a rare edge case 3NF misses):

| Student | Subject | Teacher |
|---------|---------|---------|
| Ravi    | Maths   | Mr. A   |
| Ravi    | Science | Mr. B   |
| Priya   | Maths   | Mr. A   |

Assume: each Teacher teaches only one Subject, but a Subject can have multiple Teachers, and (Student, Subject) is the primary key. Here `Teacher → Subject` is a valid dependency, but `Teacher` is **not** a candidate key → violates BCNF (even though it's already in 3NF).

- **Fix**: Split further so that every determinant is a candidate key.

**StudentTeacher**

| Student | Teacher |
|---------|---------|
| Ravi    | Mr. A   |
| Ravi    | Mr. B   |
| Priya   | Mr. A   |

**TeacherSubject**

| Teacher | Subject |
|---------|---------|
| Mr. A   | Maths   |
| Mr. B   | Science |

- **Anomaly removed**: The subtle redundancy/update anomaly caused by a non-candidate-key determinant is gone.

---

### Quick summary — which NF removes which anomaly

| Normal Form | Fixes                                             | Still Prone To                          |
|-------------|----------------------------------------------------|------------------------------------------|
| 1NF         | Non-atomic values, repeating groups               | Insert, Update, Delete anomalies (redundancy) |
| 2NF         | Partial dependency (redundancy from composite key) | Transitive dependency issues              |
| 3NF         | Transitive dependency                              | Rare anomalies from overlapping candidate keys |
| BCNF        | Anomalies from non-candidate-key determinants       | (Practically anomaly-free for most cases) |

**3NF vs BCNF**: BCNF handles rare edge cases where a table is in 3NF but still has anomalies due to overlapping/multiple candidate keys — in most real interview scenarios, 3NF is "good enough," but know BCNF exists for stricter cases.

**Example flow:** Unnormalized table (repeating columns) → 1NF (split repeating data into rows) → 2NF (remove partial dependency by splitting tables) → 3NF (remove transitive dependency by splitting further) → BCNF (fix overlapping candidate key issues).

---

### Denormalization

- **What**: The process of intentionally introducing redundancy into a normalized database — merging tables back together or duplicating columns — to improve read performance.
- **Why**: Normalization reduces redundancy but increases the number of JOINs needed to fetch data. Too many JOINs on large tables can slow down reads. Denormalization trades some redundancy for faster reads.
- **When to use**: Reporting/analytics systems, read-heavy applications (e.g., dashboards), or when JOIN cost outweighs the storage/consistency cost.

**Example:** Instead of joining `Orders` and `Customers` every time to show a customer's name on an invoice, you might store `CustomerName` directly inside the `Orders` table too — even though it's already in `Customers`.

| OrderID | CustomerID | CustomerName | Amount |
|---------|------------|--------------|--------|
| 101     | C1         | Ravi         | 5000   |

This avoids a JOIN on every query, but now if the customer's name changes, it must be updated in both places.

**Normalization vs Denormalization:**

| Aspect       | Normalization                     | Denormalization                  |
|--------------|-------------------------------------|-----------------------------------|
| Goal         | Reduce redundancy, avoid anomalies | Improve read/query performance    |
| Redundancy   | Minimal                            | Intentionally increased           |
| Writes       | Faster, safer (update in one place)| Slower/riskier (update in many places) |
| Reads        | Slower (more JOINs)                | Faster (fewer JOINs)              |
| Used in      | OLTP (transactional systems)       | OLAP (reporting/analytics systems)|

**Trade-off to remember**: Normalization favors data integrity and write efficiency; Denormalization favors read speed at the cost of redundancy and update complexity.

---

## 5. Transactions ⭐⭐⭐

- **Transaction**: A sequence of operations treated as a single unit of work.

**ACID Properties:**
- **Atomicity**: All operations succeed, or none do (all-or-nothing).
- **Consistency**: DB moves from one valid state to another, respecting rules/constraints.
- **Isolation**: Concurrent transactions don't interfere with each other.
- **Durability**: Once committed, changes persist even after a crash.

- **COMMIT**: Saves changes permanently.
- **ROLLBACK**: Undoes changes since last commit.
- **SAVEPOINT**: A checkpoint within a transaction you can roll back to (without undoing everything).
- **Transaction states**: Active → Partially Committed → Committed / Failed → Aborted.

**Bank example:** Transferring ₹500 from A to B = debit A + credit B. Atomicity ensures both happen or neither; Durability ensures once confirmed, it's saved even if the system crashes right after.

---

## 6. Concurrency Control ⭐⭐

**Problems:**
- **Dirty Read**: Reading uncommitted data from another transaction that might later be rolled back.
- **Non-repeatable Read**: Reading the same row twice in a transaction gives different results because another transaction updated it in between.
- **Phantom Read**: Re-running the same query returns new rows because another transaction inserted data in between.
- **Lost Update**: Two transactions update the same data, and one update overwrites the other.

**Locks:**
- **Locking**: Mechanism to prevent conflicting access to data.
- **Shared Lock (S)**: Multiple transactions can read, none can write.
- **Exclusive Lock (X)**: Only one transaction can read/write; blocks others.

**Isolation Levels** (weakest → strongest):
- **Read Uncommitted**: Allows dirty reads.
- **Read Committed**: Prevents dirty reads.
- **Repeatable Read**: Prevents dirty + non-repeatable reads.
- **Serializable**: Prevents all — dirty, non-repeatable, and phantom reads (strictest, slowest).

**Deadlock:**
- **What**: Two transactions wait on each other's locks forever (T1 waits for T2's resource, T2 waits for T1's).
- **Handling idea**: DB detects the cycle and aborts one transaction (timeout or wait-for-graph detection).

---

## 7. Indexing ⭐⭐⭐

- **Index**: A data structure that speeds up data retrieval (like a book's index).
- **Why**: Avoids full table scans; speeds up SELECT/WHERE/JOIN queries.
- **B+ Tree**: Balanced tree structure most DBs use for indexes — all data at leaf level, leaves linked for fast range queries.
- **Why B+ Tree**: Efficient for both exact lookups and range queries; keeps tree shallow for large data (fast disk access).
- **Clustered Index**: Determines the physical order of data in the table; only one per table (often the PK).
- **Non-clustered Index**: Separate structure pointing to actual data rows; a table can have many.

**Pros:** Faster reads/searches.
**Cons:** Slower INSERT/UPDATE/DELETE (index must be updated too), extra storage space.

**Why not index every column?** Each index adds overhead on every write operation and consumes storage — so index only columns frequently used in WHERE/JOIN/ORDER BY.

---

## 8. Joins ⭐⭐⭐

- **INNER JOIN**: Only matching rows from both tables.
- **LEFT JOIN**: All rows from left table + matched rows from right (unmatched = NULL).
- **RIGHT JOIN**: All rows from right table + matched rows from left (unmatched = NULL).
- **FULL OUTER JOIN**: All rows from both tables; unmatched sides filled with NULL.
- **SELF JOIN**: A table joined with itself (e.g., employee-manager relationship).
- **CROSS JOIN**: Cartesian product — every row of table A with every row of table B.

```sql
SELECT e.name, d.dept_name
FROM Employee e
INNER JOIN Department d ON e.dept_id = d.dept_id;
```

- **JOIN vs Subquery**: JOIN combines tables directly in one result set; subquery nests a query inside another, often used for filtering, not combining columns.
- **Joining 3 tables**: Just chain JOIN clauses with proper ON conditions between each pair.

---

## 9. SQL Command Types

- **DDL** (Data Definition): CREATE, ALTER, DROP, TRUNCATE — define/modify structure.
- **DML** (Data Manipulation): INSERT, UPDATE, DELETE — modify data.
- **DQL** (Data Query): SELECT — retrieve data.
- **DCL** (Data Control): GRANT, REVOKE — manage permissions.
- **TCL** (Transaction Control): COMMIT, ROLLBACK, SAVEPOINT — manage transactions.

**Key differences:**
- **DELETE vs TRUNCATE vs DROP**: DELETE removes rows (can filter, logged, rollback-able), TRUNCATE removes all rows fast (can't filter, minimal logging), DROP removes the entire table structure.
- **WHERE vs HAVING**: WHERE filters rows before grouping; HAVING filters groups after GROUP BY/aggregation.
- **UNION vs UNION ALL**: UNION removes duplicates; UNION ALL keeps all rows (faster).

---

## 10. Database Objects

- **View**: Virtual table from a saved query; simplifies complex queries, adds a security layer.
- **Stored Procedure**: Precompiled set of SQL statements that can perform actions (INSERT/UPDATE/etc.), callable by name.
- **Function**: Similar to a procedure but must return a value, usable inside SQL expressions.
- **Trigger**: Code that auto-executes on an event (INSERT/UPDATE/DELETE) on a table.
- **Cursor**: Used to process query results row-by-row (loop through a result set).

**Differences:**
- **Procedure vs Function**: Function must return a value and can be used in SELECT; Procedure may or may not return a value and is called separately.
- **View vs Table**: Table stores actual data; View is a saved query that shows data dynamically without storing it.

---

## 11. ER Model

- **Entity**: An object (e.g., Customer, Product).
- **Attribute**: A property of an entity (e.g., name, price).
- **Relationship**: How entities are connected (e.g., "places").
- **Strong vs Weak Entity**: Strong has its own PK; Weak depends on a strong entity for identity.

**Relationship types:**
- **One-to-One**: One entity instance relates to exactly one of another (e.g., Person ↔ Passport).
- **One-to-Many**: One entity relates to many others (e.g., Customer → many Orders).
- **Many-to-Many**: Many entities relate to many others (e.g., Students ↔ Courses).

**Example:** Customer → *places* → Order (One-to-Many: one customer can place many orders).

---

## 12. SQL — Must Practice ⭐⭐⭐

**Core syntax to be fluent in:**
- SELECT / WHERE, ORDER BY, GROUP BY, HAVING
- Aggregates: COUNT, SUM, AVG, MIN, MAX
- DISTINCT, CASE WHEN
- All JOIN types
- Subqueries and **Correlated Subquery** (inner query depends on outer query's row, runs repeatedly)
- **CTE** (`WITH temp AS (...) SELECT ...`) — makes complex queries readable
- **Window Functions**: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()` — assign ranking/numbering without collapsing rows
- **PARTITION BY** — resets window function calculation per group

**Common patterns to practice:**
- Nth highest salary (use `DENSE_RANK()` or subquery with `LIMIT`/`OFFSET`)
- Top N per group (use `ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)`)
- Employees earning more than their manager (self join)
- Find/delete duplicate records (GROUP BY + HAVING COUNT > 1, or ROW_NUMBER)
- Department-wise counts/highest salary (GROUP BY + aggregate)
- Customers with no orders (LEFT JOIN + WHERE right side IS NULL)
- Latest record per customer (window function or correlated subquery)
- Running total (`SUM(...) OVER (ORDER BY date)`)
- Consecutive-day/activity problems (window functions comparing row to previous row)

---

## Priority Order (if short on time)

**Tier 1 (must be strong):** Normalization → Keys → ACID/Transactions → Joins + SQL → Indexing

**Tier 2 (good understanding):** Concurrency + Isolation Levels → Constraints → ER Model

**Tier 3 (quick definitions only):** DBMS/RDBMS basics → Schema/Instance/Degree/Cardinality → Views/Triggers/Cursors/Procedures/Functions → SQL command types

**Skip entirely:** Relational algebra/calculus, serializability proofs, precedence graphs, timestamp protocols, 2PL internals, recovery/log-based algorithms, RAID/file organization, query optimizer internals, distributed DBs.

---

*Once this is solid, shift remaining prep time to: projects, SQL coding practice, OOP, OS, DSA, system design.*
