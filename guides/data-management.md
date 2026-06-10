# CS 35L Lecture 8 - Data Management

Instructor: Tobias Durschmid, Assistant Teaching Professor, Computer Science Department
Course: CS 35L Software Construction

---

## 1. Why Do We Need a Database Management System (DBMS)?

When storing data, **things can go wrong**:
- **Power Outage** during write operations
- **Multiple simultaneous edits** to the same data

A DBMS provides the following benefits:
- **Redundancy** prevents data loss in the case of disk failures
- **Indexing** optimizes your Data Read speeds
- **Query Optimization** optimizes the speed of your queries

Architecture flow:
```
Your Application  ->  DBMS  ->  Storage (disk)
```

The DBMS sits between your application and the underlying storage, mediating reads/writes so the application doesn't have to worry about low-level concerns like failure handling, concurrency, indexing, and query speed.

---

## 2. SQL (Structured Query Language)

**SQL allows you to read and write data for most DBMS.**

Key properties of SQL:
- **SQL is declarative**
  - You specify the operation, the DBMS figures out **how** to do this efficiently.
- The **industry-standard** for almost all DBMS.
- Allows you to **swap out one DBMS for another** (mostly).
- Other query languages are **similar to SQL**.

Architecture flow with SQL:
```
Your Application  --SQL Query-->  DBMS  ->  Storage
```

**Course note**: In this course, you do **NOT** have to learn SQL syntax. SQL queries are shown only for illustrative purposes. But you are welcome to use SQL in your projects.

---

## 3. Database Structures Are Modeled Using ER Diagrams

**ER (Entity-Relationship) Diagrams** model database structures.

### Example ER Diagram (Student - is enrolled - Course)

- **Entities (rectangles)**: `Student`, `Course`
- **Relationship (diamond)**: `is enrolled`
- **Cardinality**: `N` on the Student side, `M` on the Course side (a many-to-many relationship: each student may enroll in many courses, and each course may have many students)
- **Attributes (ovals)** attached to entities:
  - Student: `UID` (primary key, underlined), `Name`
  - Course: `ID` (primary key, underlined), `Instructor`, `Quarter` (also part of primary key - underlined)

### Primary Keys
- **A primary key** is **underlined** in the ER diagram.
- It **specifies the properties that make a data entry unique** among others.
- It's the **"address"** of that entry.

A primary key may be composed of multiple attributes (composite key). For example, the Course primary key uses both `id` and `quarter`, since "35L" is offered in multiple quarters but each (id, quarter) combination is unique.

Discussion prompt from lecture: "Which attributes do we need to store? Talk to your neighbor."

---

## 4. Relational Database Management Systems (RDBMS) Store Data in Tables (Relations)

**Examples of RDBMS**: Oracle DB, IBM DB2, MySQL, SQLite, PostgreSQL, ...

Each entity/relationship in the ER diagram becomes a **table (relation)** in the database.

### Example tables (from the ER diagram above)

**Student** table:
| name      | uid (PK) |
|-----------|----------|
| Jon Doe   | 12345    |
| Jane Doe  | 23456    |

**IsEnrolled** table (relationship table linking Student to Course):
| uid (PK) | quarter (PK) | course_id (PK) |
|----------|--------------|----------------|
| 12345    | Fall 2025    | 35L            |
| 12345    | Fall 2025    | 143            |

**Course** table:
| id (PK) | quarter (PK) | instructor       |
|---------|--------------|------------------|
| 35L     | Fall 2025    | Tobias Durschmid |
| 143     | Fall 2025    | Remy Wang        |
| 32      | Fall 2025    | David Smallberg  |

### SQL `CREATE TABLE` for these tables

**Student**:
```sql
CREATE TABLE Student (
    uid INTEGER NOT NULL PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```

**Course**:
```sql
CREATE TABLE Course (
    id VARCHAR(50) NOT NULL,
    quarter VARCHAR(20) NOT NULL,
    instructor VARCHAR(100),
    PRIMARY KEY (id, quarter)
);
```

**IsEnrolled** (uses foreign keys to link to Student and Course):
```sql
CREATE TABLE IsEnrolled (
    uid INTEGER NOT NULL,
    course_id VARCHAR(50) NOT NULL,
    quarter VARCHAR(20) NOT NULL,
    PRIMARY KEY (uid, course_id, quarter),
    FOREIGN KEY (uid)
        REFERENCES Student(uid)
    FOREIGN KEY (course_id, quarter)
        REFERENCES Course(id, quarter)
);
```

### Foreign Keys

**A foreign key is a reference to the primary key of another data element to link it consistently.**

In the `IsEnrolled` table:
- `uid` is a foreign key referencing `Student(uid)`.
- `(course_id, quarter)` is a composite foreign key referencing `Course(id, quarter)`.

This ensures **referential integrity** - you cannot insert an `IsEnrolled` row referencing a student or course that doesn't exist.

---

## 5. DBMS Support a Large Variety of Operations

Example query types over the Student / IsEnrolled / Course tables:
- "Give me the **names of all students** who have taken 35L"
- "**Count** all students who have taken a course with Remy Wang"
- "**For each instructor**, count all students who have taken a course with them"

These illustrate the kinds of expressive declarative queries that DBMS support.

---

## 6. Database Operation: Join (R JOIN S)

**Join combines columns from one or more tables into a new table**, based on one or more related columns between them.
- Looks for **matching values** in the related columns to combine the rows.

### Example: `Student JOIN IsEnrolled JOIN Course`

Starting tables:
- **Student**: (Jon Doe, 12345), (Jane Doe, 23456)
- **IsEnrolled**: (12345, Fall 2025, 35L), (12345, Fall 2025, 143)
- **Course**: (35L, Fall 2025, Tobias Durschmid), (143, Fall 2025, Remy Wang), (32, Fall 2025, David Smallberg)

Result of `Student JOIN IsEnrolled JOIN Course`:
| name    | uid   | quarter   | course_id | instructor       |
|---------|-------|-----------|-----------|------------------|
| Jon Doe | 12345 | Fall 2025 | 35L       | Tobias Durschmid |
| Jon Doe | 12345 | Fall 2025 | 110       | Remy Wang        |

(Note: in the lecture's join result, `course_id = 110` appears in the second row - this matches what was shown on the slide.)

### Types of Joins

**Note**: Whether the result also contains partial results with null values depends on the **type of join used**:
- `inner join` - only rows with matching values on both sides; non-matching rows are dropped.
- `outer join` (full outer) - includes all rows from both sides; non-matching cells become null.
- `left outer join` - keeps all rows from the left table; right-side cells are null if no match.
- `right outer join` - keeps all rows from the right table; left-side cells are null if no match.

References:
- https://www.geeksforgeeks.org/dbms/joins-in-dbms/
- https://www.w3schools.com/sql/sql_join.asp

---

## 7. Database Operation: Selection (sigma)

**Selection uses a Boolean predicate on rows to get a sub-table.**

It filters which **rows** appear in the result, based on a condition. Columns are unchanged.

### Example

Input: `Student JOIN IsEnrolled JOIN Course`:
| name    | uid   | quarter   | course_id | instructor       |
|---------|-------|-----------|-----------|------------------|
| Jon Doe | 12345 | Fall 2025 | 35L       | Tobias Durschmid |
| Jon Doe | 12345 | Fall 2025 | 143       | Remy Wang        |

Selection: `sigma_{course_id = 35L} (Student JOIN IsEnrolled JOIN Course)`:
| name    | uid   | quarter   | course_id | instructor       |
|---------|-------|-----------|-----------|------------------|
| Jon Doe | 12345 | Fall 2025 | 35L       | Tobias Durschmid |

In SQL, selection corresponds to the `WHERE` clause.

Reference: https://www.w3schools.com/sql/sql_where.asp

---

## 8. Database Operation: Projection (Pi)

**Projection eliminates columns.**

It chooses which **columns** to keep in the result. Rows are unchanged (except possibly deduplication).

### Example

Input: `sigma_{course_id = 35L} (Student JOIN IsEnrolled JOIN Course)`:
| name    | uid   | quarter   | course_id | instructor       |
|---------|-------|-----------|-----------|------------------|
| Jon Doe | 12345 | Fall 2025 | 35L       | Tobias Durschmid |

Projection: `Pi_{name} (sigma_{course_id = 35L} (Student JOIN IsEnrolled JOIN Course))`:
| name    |
|---------|
| Jon Doe |

In SQL, projection corresponds to the column list in `SELECT`.

Reference: https://www.w3schools.com/sql/sql_select.asp

---

## 9. Example Query 1: "Give me the Names of all Students who have taken 35L"

### Relational Algebra
```
Pi_{name} ( sigma_{course_id = 35L} ( Student JOIN IsEnrolled ) )
```

### Result
| name    |
|---------|
| Jon Doe |

### SQL Query
```sql
SELECT S.name
FROM Student AS S JOIN IsEnrolled AS E
    ON S.uid = E.uid
WHERE E.course_id = '35L';
```

### Mapping SQL <-> Relational Algebra
- `SELECT S.name` - **Projection** "Give me the names"
- `FROM Student AS S JOIN IsEnrolled AS E ON S.uid = E.uid` - **Join** to link "all students" with course name
- `WHERE E.course_id = '35L'` - **Selection** "who have taken 35L"

Reference: https://www.w3schools.com/sql/default.asp

---

## 10. Example Query 2: "Count all Students who have taken a course with Remy Wang"

This query is broken into 3 steps.

### Updated example data

**Student**: (Jon Doe, 12345), (Jane Doe, 23456)

**IsEnrolled**:
| uid   | quarter   | course_id |
|-------|-----------|-----------|
| 12345 | Fall 2025 | 35L       |
| 12345 | Fall 2025 | 143       |
| 23456 | Fall 2024 | 143       |

**Course**:
| id  | quarter   | instructor       |
|-----|-----------|------------------|
| 35L | Fall 2025 | Tobias Durschmid |
| 143 | Fall 2025 | Remy Wang        |
| 143 | Fall 2024 | Remy Wang        |

### Step 1: Selection by Instructor Name

`sigma_{instructor = Remy Wang} (Course)`:
| id  | quarter   | instructor |
|-----|-----------|------------|
| 143 | Fall 2025 | Remy Wang  |
| 143 | Fall 2024 | Remy Wang  |

### Step 2: Join with Enrollment to link UID with instructor name

`IsEnrolled JOIN sigma_{instructor = Remy Wang} (Course)`:
| uid   | quarter   | course_id | instructor |
|-------|-----------|-----------|------------|
| 12345 | Fall 2025 | 143       | Remy Wang  |
| 23456 | Fall 2024 | 143       | Remy Wang  |

### Step 3: COUNT DISTINCT on UID

Result:
| Result |
|--------|
| 2      |

### SQL Query
```sql
SELECT COUNT(DISTINCT E.uid) AS Result
FROM IsEnrolled AS E JOIN Course AS C
    ON E.course_id = C.id
    AND E.quarter = C.quarter
WHERE C.instructor = 'Remy Wang';
```

**Why `COUNT(DISTINCT E.uid)`?** If a student took multiple Remy Wang courses, a plain `COUNT(*)` would double-count them. `DISTINCT` ensures each student is counted only once.

Reference: https://www.w3schools.com/sql/sql_count.asp

---

## 11. Example Query 3: "For Each Instructor, Count all Students who have taken a course with them"

### Relational Algebra
```
gamma_{instructor, agg(COUNT)} ( IsEnrolled JOIN Course )
```

Here `gamma` is the grouping/aggregation operator: group by `instructor`, then aggregate with `COUNT`.

### Result
| Instructor       | Students |
|------------------|----------|
| Tobias Durschmid | 1        |
| Remy Wang        | 2        |

### SQL Query
```sql
SELECT C.instructor, COUNT(DISTINCT E.uid) AS students
FROM IsEnrolled AS E JOIN Course AS C
    ON E.course_id = C.id
    AND E.quarter = C.quarter
GROUP BY C.instructor;
```

### GROUP BY
**`GROUP BY` aggregates all rows who have the same value in the given column (instructor).**

When combined with an aggregate function like `COUNT`, `SUM`, `AVG`, `MIN`, or `MAX`, each group produces a single output row.

Reference: https://www.w3schools.com/sql/sql_groupby.asp

---

## 12. Transactions

A **transaction** is **a single, logical unit of work** - a group of database operations that must be treated as one atomic block.

### Case Study: Bank Transfer

Setup - two accounts in an `Accounts` table:

| ID | Balance |
|----|---------|
| A  | 2000    |

| ID | Balance |
|----|---------|
| B  | 1000    |

**Goal**: Transfer 100 USD from Account A to Account B.

### SQL Transaction
```sql
BEGIN TRANSACTION;
    UPDATE Accounts
        SET Balance = Balance - 100
        WHERE ID = 'A';
    UPDATE Accounts
        SET Balance = Balance + 100
        WHERE ID = 'B';
COMMIT;
```

### Breakdown of the SQL
- `BEGIN TRANSACTION;` - the following steps are in **one Transaction**.
- `UPDATE Accounts` - specifies the **table** to be updated.
- `SET Balance = Balance - 100` - specification of the **change**.
- `WHERE ID = 'A';` - specification of the **data rows** to be changed.
- `COMMIT;` - **saves changes** to the database (ends the transaction).

If we want to abort partway through, we would issue a `ROLLBACK` instead of `COMMIT` (implied by ACID atomicity below).

---

## 13. ACID Properties of Transactions

Transactions in a (traditional) DBMS guarantee four properties, summarized by the acronym **ACID**:
- **A**tomicity
- **C**onsistency
- **I**solation
- **D**urability

Reference: https://www.geeksforgeeks.org/dbms/acid-properties-in-dbms/

### A - Atomicity

**Definition**: A transaction is treated as a **single, all-or-nothing unit**. Either all operations succeed, or none of them do; if any part fails, the entire transaction is rolled back.

**Bank Transfer case study**: Under no conditions does the database end in a state in which account A's balance has been changed without also changing account B's balance.

### C - Consistency

**Definition**: A transaction must bring the database **from one valid state to another**. This ensures that data adheres to all rules and constraints, preventing invalid data from being written.

**Bank Transfer case study**: Under no conditions does any changed account violate any of the specified rules (e.g. all balances have to be integers, all balances are non-negative). Otherwise, the transaction is declined.

### I - Isolation

**Definition**: **Concurrent transactions do not interfere with each other**. The changes made by one transaction are not visible to other transactions until the first one is complete.

**Bank Transfer case study**: If simultaneously we calculate the total balance of the entire bank, we either read balances before the transfer or after the transfer, but never in the intermediate state in which only one balance has been changed.

### D - Durability

**Definition**: Once a transaction is committed, its **changes are permanent** and will **survive any system failures, such as power outages**.

**Bank Transfer case study**: A power outage happens right after we commit the bank transfer. Fortunately, the system recovers the transaction from the logs to ensure that all balance changes are permanent.

---

## 14. NoSQL (Not Only SQL) - Alternative to Traditional Relational DBMS

**Examples of NoSQL DBMS**: MongoDB, Redis, Apache Cassandra.

### Definition
A **non-relational database approach** that stores and manages data in **flexible, non-tabular formats** like:
- **Documents** (e.g., JSON-like documents - MongoDB)
- **Key-value pairs** (e.g., Redis)
- **Wide-columns** (e.g., Apache Cassandra)
- **Graphs**

### RDBMS vs. NoSQL on ACID

- **RDBMS guarantee ACID**.
- **NoSQL doesn't always guarantee ACID**:
  - Many NoSQL databases **relax the strict isolation levels to maximize performance**.
  - Many NoSQL databases use a **weaker consistency model (Eventual Consistency)**.
  - Often **depends on your configuration** of the system.

### Eventual Consistency

A weaker consistency model used in many NoSQL systems: after a write, replicas across the system will eventually converge to the same value, but reads may temporarily see stale data.

### When to Use Which

| RDBMS | NoSQL |
|-------|-------|
| Better for **well-structured data with transactions** | Better for **big data that is less well-structured** and where **transactions are less important** |

---

## 15. Case Study for Distributed Database Systems: ATMs (Automated Teller Machines)

A distributed system of ATMs and Bank Servers connected via a network illustrates three properties:

- **Consistency** - ATMs always show you the **correct current balance** of your account.
- **Availability** - ATMs allow you to **withdraw money at any given time**.
- **Partition Tolerance** - ATMs allow you to withdraw money from **different physical locations** (and continue to work when servers are partitioned from each other).

---

## 16. Properties of Distributed Database Systems

Formal definitions:

### Consistency
**Every read receives the most recent write or an error.**

### Availability
**Every read receives a (non-error) response.**

### Partition Tolerance
**The system continues to operate despite network partitions.**

A "network partition" is when the network splits into groups of nodes that can no longer communicate with each other.

Discussion prompt from lecture: "How can we design a system that supports all three? Talk to your neighbor(s)."

---

## 17. CAP Theorem: You Can Pick Two at Most

**The CAP Theorem** states that for a distributed database system, you can guarantee **at most two** of the three properties (Consistency, Availability, Partition Tolerance) simultaneously.

The three properties form a Venn diagram with three overlapping circles labeled:
- **C**onsistency
- **A**vailability
- **P**artition Tolerance

Restated:
- **Consistency** - Every read receives the most recent write or an error.
- **Availability** - Every read receives a (non-error) response.
- **Partition Tolerance** - The system continues to operate despite network partitions.

---

## 18. Different DBMS Make Different Trade-offs

The three possible "pick two" combinations correspond to different classes of database systems:

### CA: Traditional Single Node Relational DBMS
- Guarantees **Consistency** and **Availability** but not Partition Tolerance.
- Examples: **MySQL, Oracle, PostgreSQL**.
- Because there is only one node, partitioning isn't a concern - but the system cannot scale horizontally across the network in the same way.

### AP: Eventually-Consistent DBMS
- Guarantees **Availability** and **Partition Tolerance** but sacrifices strong Consistency (uses eventual consistency instead).
- Examples: **Most NoSQL databases - Apache Cassandra, Amazon DynamoDB, CouchDB**.

### CP: MongoDB (when configured for strong consistency)
- Guarantees **Consistency** and **Partition Tolerance** but sacrifices Availability under partitions.
- Example: **MongoDB** (when configured for **strong consistency**, e.g., using a majority write concern).
- During a network partition, it will **sacrifice availability in the minority partition** to ensure the remaining available nodes hold consistent data.

---

## 19. SQL Is Optional in This Course

SQL is optional. If you'd like to use SQL in your project, the recommended resource is **Remy's Tutorial**:
- https://remy.wang/cs143/notes/sql/sql.html

The instructor is also working on a SQL tutorial, but it is not ready yet.

---

## 20. Summary / Exit Ticket Questions

The lecture ends with three exit-ticket questions:
1. **Please summarize three database operations you learned today.** (e.g., Join, Selection, Projection - or also Group By / aggregation.)
2. **Please describe in your own words what the ACID properties are and why they are important for data management.**
3. **Please leave any questions that you have about today's material** and things that are still unclear or confusing to you (if none, simply write N/A).

---

## Quick Reference Cheat Sheet

### Relational Algebra Operators
| Symbol | Operator   | What it does                        | SQL equivalent          |
|--------|------------|-------------------------------------|-------------------------|
| JOIN   | Join       | Combines tables on matching columns | `JOIN ... ON ...`       |
| sigma  | Selection  | Filters rows by a predicate         | `WHERE ...`             |
| Pi     | Projection | Keeps only chosen columns           | column list in `SELECT` |
| gamma  | Grouping/Aggregation | Group rows + aggregate    | `GROUP BY ...`          |

### Join Types
- **Inner join** - only matching rows.
- **Outer join (full)** - all rows from both sides, nulls for non-matches.
- **Left outer join** - all rows from the left table.
- **Right outer join** - all rows from the right table.

### Key SQL Keywords Seen
- `CREATE TABLE`, `PRIMARY KEY`, `FOREIGN KEY ... REFERENCES`
- `SELECT`, `FROM`, `JOIN ... ON`, `WHERE`, `GROUP BY`
- `COUNT(...)`, `COUNT(DISTINCT ...)`, `AS` (alias)
- `BEGIN TRANSACTION`, `UPDATE ... SET ... WHERE`, `COMMIT`

### ACID
- **A**tomicity - all-or-nothing.
- **C**onsistency - only valid states.
- **I**solation - concurrent transactions don't interfere.
- **D**urability - committed changes survive failures.

### CAP
- **C**onsistency - most recent write or error.
- **A**vailability - non-error response always.
- **P**artition Tolerance - keep running through network splits.
- Pick **at most two**. Common categories: **CA** (single-node RDBMS), **AP** (most NoSQL, eventual consistency), **CP** (MongoDB with strong consistency).


---

# Appendix: Lecture 8 Slides (raw extracted text)

The following is the full extracted text of Lecture 8. Preserved verbatim so no information is lost.

```text
CS 35L Software
Construction
Lecture 8 ­ Data
Management
Assistant Teaching Professor
Computer Science Department
Why do we need a Database Management System
(DBMS)?
· When storing data things can go wrong            Your Application
   · Power Outage during write operations                DBMS
   · Multiple simultaneous edits to the same data
· Redundancy prevents data loss
  in the case of disk failures
· Indexing optimizes your Data Read speeds
· Query Optimization optimizes the
  speed of your queries
SQL (Structured Query Language)
Allows you to read and write data for most DBMS
· SQL is declarative             In this course, you     Your Application
   · You specify the operation,  do NOT have to            SQL Query
     the DBMS figures out        learn SQL syntax.             DBMS
     how to do this efficiently  We only show SQL
                                 queries for
· The industry-standard          illustrative purposes.
  for almost all DBMS            But you are
                                 welcome to
· Allows you to swap out one     use SQL in your
  DBMS for another (mostly)      projects.
· Other query languages are
similar to SQL
Database Structures                      Which attributes do we
are Modeled Using ER Diagrams            need to store? Talk to
                                         your neighbor
                         N  is enrolled  M
Student                                        Course
 UID  A primary key is                                               ID
Name  underlined and specifies                                   Instructor
      the properties that make a                                  Quarter
      data entry unique among
      others. It's the "address" of
      that entry
      CS 35L Software Construction: Lecture 8 ­ Data Management              4
      Tobias Dürschmid
Relational Database Management Systems (e.g., Oracle DB, IBM
                                                                                                                                              DB2, MySQL, SQLite,
(RDBMS) Store Data in Tables (Relations) PostgreSQL, ...)
                                  N  is enrolled  M
         Student                                        Course
      UID                                                                ID
      Name                                                               Instructor
Student         IsEnrolled                        Course                     Quarter
name     uid    uid quarter course_id             id quarter             instructor
                12345 Fall 2025 35L               35L Fall 2025          Tobias Dürschmid
Jon Doe 12345   12345 Fall 2025 143               143 Fall 2025          Remy Wang
                                                  32 Fall 2025           David Smallberg
Jane Doe 23456
              CS 35L Software Construction: Lecture 8 ­ Data Management                    5
              Tobias Dürschmid
Relational Database Management Systems (e.g., Oracle DB, IBM
                                                                                                                                              DB2, MySQL, SQLite,
(RDBMS) Store Data in Tables (Relations) PostgreSQL, ...)
Student                 IsEnrolled                                      Course
name     uid            uid quarter course_id id quarter instructor
Jon Doe 12345           12345 Fall 2025 35L                             35L Fall 2025 Tobias Dürschmid
Jane Doe 23456          12345 Fall 2025 143                             143 Fall 2025 Remy Wang
    SQL Query                                                           32 Fall 2025 David Smallberg
CREATE TABLE Student (                                                    SQL Query
    uid INTEGER NOT NULL PRIMARY KEY,                                   CREATE TABLE Course (
                                                                           id VARCHAR(50) NOT NULL,
    name VARCHAR(100) NOT NULL                                             quarter VARCHAR(20) NOT NULL,
);                      SQL Query
                        CREATE TABLE IsEnrolled (                           instructor VARCHAR(100),
                        uid INTEGER NOT NULL,                               PRIMARY KEY (id, quarter)
                        course_id VARCHAR(50) NOT NULL,                 );
                        quarter VARCHAR(20) NOT NULL,
                        PRIMARY KEY (uid, course_id, quarter),
                        FOREIGN KEY (uid)                               A foreign key is a reference to
                           REFERENCES Student(uid)
                        FOREIGN KEY (course_id, quarter) the primary key of another data
               CS 35L SoftwaRreEFCEoRnEsNtCrEuSctiCoonu:rLseec(tiudr,e  8qu­aDrtaetar )Manaegleemmeennt t  to  link  it  consistently  6
               Tobias Dür)s;chmid
DBMS Support a Large Variety of Operations
Student         IsEnrolled             Course
name     uid    uid quarter course_id  id quarter                        instructor
                12345 Fall 2025 35L    35L Fall 2025                     Tobias Dürschmid
Jon Doe 12345   12345 Fall 2025 143    143 Fall 2025                     Remy Wang
                                       32 Fall 2025                      David Smallberg
Jane Doe 23456
e.g.:
· "Give me the names of all students who have taken 35L"
· "Count all students who have taken a course with Remy Wang"
· "For each instructor, count all students who have taken a course with them"
              CS 35L Software Construction: Lecture 8 ­ Data Management                    7
              Tobias Dürschmid
Database Operation: Join (RS)
Student         IsEnrolled             Course
name     uid    uid quarter course_id  id quarter                                                                   instructor
                12345 Fall 2025 35L    35L Fall 2025                                                                Tobias Dürschmid
Jon Doe 12345   12345 Fall 2025 143    143 Fall 2025                                                                Remy Wang
                                       32 Fall 2025                                                                 David Smallberg
Jane Doe 23456
· Join combines columns from one or more tables into a new table, based on
  one or more related columns between them
   · Looks for matching values in the related columns to combine the rows
Read more here: hhtttpst:p//wsw:w//.gweekwsfowrge.egkse.oreg/kdbsmfs/ojoirngs-ine-debmks/s.org/dbms/joins-in-dbms/
              CS 35L Software Construction: Lecture 8 ­ Data Management                                                               8
              Tobias Dürschmid
Database Operation: Join (RS)
Student              IsEnrolled                                                                       Course
name     uid         uid quarter course_id                                                            id quarter     instructor
                     12345 Fall 2025 35L                                                              35L Fall 2025  Tobias Dürschmid
Jon Doe 12345        12345 Fall 2025 143                                                              143 Fall 2025  Remy Wang
                                                                                                      32 Fall 2025   David Smallberg
Jane Doe 23456
Student  IsEnrolled  Course
name            uid             quarter                                                               course_id      instructor
                                                                                                      35L            Tobias Dürschmid
Jon Doe         12345           Fall 2025                                                             110            Remy Wang
Jon Doe         12345           Fall 2025
Note: Whether the result also contains partial results with null values depends on the type of join used.
(inner join, outer join, left outer join, right outer join)
Read more here: hhtttpst:p//wsw:w//.ww3swchowols..cwom3/sqsl/csqlh_jooin.oaslps.com/sql/sql_join.asp
              CS 35L Software Construction: Lecture 8 ­ Data Management                                                                9
              Tobias Dürschmid
Database Operation: Selection 
Student  IsEnrolled  Course
name     uid               quarter                                                                      course_id  instructor
                                                                                                        35L        Tobias Dürschmid
Jon Doe  12345             Fall 2025                                                                    143        Remy Wang
Jon Doe  12345             Fall 2025
· Selection uses a Boolean predicate on rows to get a sub-table:
_= (Student  IsEnrolled  Course)
name     uid               quarter                                                                      course_id  instructor
                                                                                                                   Tobias Dürschmid
Jon Doe  12345             Fall 2025                                                                    35L
Read more here: hhtttpst:p//wsw:w//.ww3swchowols..cwom3/sqsl/csqlh_woheorel.assp.com/sql/sql_where.asp
         CS 35L Software Construction: Lecture 8 ­ Data Management                                                                   10
         Tobias Dürschmid
Database Operation: Projection 
_= (Student  IsEnrolled  Course)
name     uid               quarter                                                                        course_id  instructor
                                                                                                                     Tobias Dürschmid
Jon Doe  12345             Fall 2025                                                                      35L
· Projection eliminates columns
 (_=(Student  IsEnrolled  Course))
 name
 Jon Doe
Read more here: hhtttpst:p//wsw:w//.ww3swchowols..cwom3/sqsl/csqlh_soeleoct.lassp.com/sql/sql_select.asp
         CS 35L Software Construction: Lecture 8 ­ Data Management                                                                     11
         Tobias Dürschmid
Query for: "Give me the Names of all Students
who have taken 35L"
Student                               IsEnrolled             Course
name                             uid  uid quarter course_id id quarter instructor
Jon Doe 12345                         12345 Fall 2025 35L    35L Fall 2025 Tobias Dürschmid
Jane Doe 23456                        12345 Fall 2025 143    143 Fall 2025 Remy Wang
· Result:                                                    32 Fall 2025 David Smallberg
(_=(Student  IsEnrolled))
name                                  SQL Query
  Jon Doe                        SELECT S.name          Projection "Give me the names"
Read more                        FROM Student AS S JOIN IsEnrolled AS E                          Join to link "all students"
about SQL here:                                                                                  with course name
hhtttptsp:/s/w:/w/ww.ww3sw.w3s                          ON S.uid = E.uid
cchhoools.ocolms./csqol/ m/sql/
ddeefafualt.uaslpt.asp           WHERE E.course_id = '35L';  Selection "who have taken 35L"
                                      CS 35L Software Construction: Lecture 8 ­ Data Management                               12
                                      Tobias Dürschmid
Query for "Count all students who have taken a
course with Remy Wang"
Student         IsEnrolled                  Course
name     uid    uid quarter course_id       id quarter                   instructor
                12345 Fall 2025 35L         35L Fall 2025                Tobias Dürschmid
Jon Doe 12345   12345 Fall 2025 143         143 Fall 2025                Remy Wang
                23456 Fall 2024 143         143 Fall 2024                Remy Wang
Jane Doe 23456
Step 1: Selection by Instructor Name
=  (Course)
id              quarter         instructor
143             Fall 2025       Remy Wang
143             Fall 2024       Remy Wang
              CS 35L Software Construction: Lecture 8 ­ Data Management                    13
              Tobias Dürschmid
Query for "Count all students who have taken a
course with Remy Wang"
Student         IsEnrolled                 Course
name     uid    uid quarter course_id      id quarter                    instructor
                12345 Fall 2025 35L        35L Fall 2025                 Tobias Dürschmid
Jon Doe 12345   12345 Fall 2025 143        143 Fall 2025                 Remy Wang
                23456 Fall 2024 143        143 Fall 2024                 Remy Wang
Jane Doe 23456
Step 2: Join with Enrollment to link UID with instructor name
IsEnrolled  =  (Course)
uid             quarter         course_id                                instructor
12345           Fall 2025       143                                      Remy Wang
23456           Fall 2024       143                                      Remy Wang
              CS 35L Software Construction: Lecture 8 ­ Data Management                    14
              Tobias Dürschmid
Query for "Count all students who have taken a
course with Remy Wang"
Student                                   IsEnrolled             Course
name                                 uid  uid quarter course_id  id quarter                          instructor
                                          12345 Fall 2025 143    35L Fall 2025                       Tobias Dürschmid
Jon Doe 12345                             23456 Fall 2024 143    143 Fall 2025                       Remy Wang
                                          12345 Fall 2025 143    143 Fall 2024                       Remy Wang
Jane Doe 23456
Step 3: COUNT DISTINCT on UID
Result                                    SQL Query
2                                    SELECT COUNT(DISTINCT E.uid) AS Result
                                     FROM IsEnrolled AS E JOIN Course AS C
Read more here                                                             ON E.course_id = C.id
hhttttpps:s//:w//wwww.ww3.swc 3sc                                          AND E.quarter = C.quarter
hhooools.lcso.mco/smql//ssqql l/sql  WHERE C.instructor = 'Remy Wang';
__ccoounut.nats.pasp
                                          CS 35L Software Construction: Lecture 8 ­ Data Management                    15
                                          Tobias Dürschmid
Query for "For each instructor, count all students
who have taken a course with them"
Student                                                        IsEnrolled           Course
name            uid                                            uid quarter course_id id quarter instructor
Jon Doe 12345                                                  12345 Fall 2025 35L  35L Fall 2025 Tobias Dürschmid
Jane Doe 23456                                                 12345 Fall 2025 143  143 Fall 2025 Remy Wang
· Result:                                                      23456 Fall 2024 143  32 Fall 2025 David Smallberg
,()(IsEnrolled  Course)
Instructor Students                                            SQL Query
Tobias          1                                              SELECT C.instructor, COUNT(DISTINCT E.uid) AS students
Dürschmid
Remy            2                                              FROM IsEnrolled AS E JOIN Course AS C
                                                                                                    ON E.course_id = C.id
Wang
                                                                                    AND E.quarter = C.quarter
Read more here
hhttttpps:s//:w//wwww.ww3.swch3osolcs.hcoomo/lssq.lc/ om/sql/  GROUP BY C.instructor;16 Group By aggregates all rows who have the
ssqql_lg_rgourpobuyp.abspy.asp
                     CS 35L Software Construction: Lecture 8 ­ Data Masnaamgeemveanlut e in the given column (instructor)
                     Tobias Dürschmid                                                                                      16
A single, logical unit of work
Transaction Case Study: Bank Transfer
Accounts  Balance               Accounts                             Balance
          2000                                                       1000
ID                              ID
A                               B
Transaction: Transfer 100 USD from Account A to Account B
BEGIN TRANSACTION; Following steps are in one Transaction            SQL Query
UPDATE Accounts                 Table to be updated
          SET Balance = Balance - 100 Specification of the change
          WHERE ID = 'A';       Specification of the data rows to be changed
UPDATE Accounts
          SET Balance = Balance + 100
          WHERE ID = 'B';
COMMIT; Saves changes to the database (ends the transaction)
          CS 35L Software Construction: Lecture 8 ­ Data Management             17
          Tobias Dürschmid
ACID Properties of Transactions
  Atomicity
A transaction is treated as a single, all-or-nothing unit. Either all
operations succeed, or none of them do; if any part fails, the entire
transaction is rolled back.
  Case Study: Bank Transfer
Under no conditions does the database end in a state in which account
A's balance has been changed without also changing account B's
balance.
Read more here: hhtttpst:p//wsw:w//.gweekwsfowrge.egkse.oreg/kdbsmfs/oacridg-peropeerktiess-i.no-dbrmgs//dbms/acid-properties-in-dbms/
ACID Properties of Transactions
  Consistency
A transaction must bring the database from one valid state to another.
This ensures that data adheres to all rules and constraints, preventing
invalid data from being written.
  Case Study: Bank Transfer
Under no conditions does any changed account violate any of the
specified rules (e.g. all balances have to be integers, all balances are
non-negative). Otherwise, the transaction is declined.
Read more here: hhtttpst:p//wsw:w//.gweekwsfowrge.egkse.oreg/kdbsmfs/oacridg-peropeerktiess-i.no-dbrmgs//dbms/acid-properties-in-dbms/
ACID Properties of Transactions
  Isolation
Concurrent transactions do not interfere with each other. The
changes made by one transaction are not visible to other transactions
until the first one is complete.
  Case Study: Bank Transfer
If simultaneously we calculate the total balance of the entire bank, we
either read balances before the transfer or after the transfer, but never in
the intermediate state in which only one balance has been changed.
Read more here: hhtttpst:p//wsw:w//.gweekwsfowrge.egkse.oreg/kdbsmfs/oacridg-peropeerktiess-i.no-dbrmgs//dbms/acid-properties-in-dbms/
ACID Properties of Transactions
   Durability
 Once a transaction is committed, its changes are permanent and will
 survive any system failures, such as power outages.
  Case Study: Bank Transfer
A power outage happens right after we commit the bank transfer.
Fortunately, the system recovers the transaction from the logs to ensure
that all balance changes are permanent.
Read more here: hhtttpst:p//wsw:w//.gweekwsfowrge.egkse.oreg/kdbsmfs/oacridg-peropeerktiess-i.no-dbrmgs//dbms/acid-properties-in-dbms/
NoSQL (Not Only SQL) (e.g., MongoDB, Redis, Apache Cassandra)
Is an Alternative to Traditional Relational DBMS
· Non-relational database approach that stores and manages data in flexible, non-
  tabular formats like documents, key-value pairs, wide-columns, or graphs
· RDBMS guarantee ACID, NoSQL doesn't always guarantee ACID
· Many NoSQL databases relax the strict isolation levels to maximize performance
· Many NoSQL databases use a weaker consistency model (Eventual Consistency)
· Often depends on your configuration of the system
   RDBMS                                 NoSQL
Better for well-structured data with  Better for big data that is less well-structured
transactions                          and where transactions are less important
Case Study for Distributed Database Systems
ATMs (Automated Teller Machines)
  Consistency                      BANK Server             BANK Server
ATMs always show you the correct                                          23
current balance of your account
  Availability
ATMs allow you to withdraw money
at any given time
  Partition Tolerance
ATMs allow you to withdraw money
from different physical locations
  How can we design a system that supports all three? Talk to your neighbor(s)
Properties of Distributed Database Systems
  Consistency
Every read receives the most recent
write or an error
  Availability
Every read receives a (non-error)
response
  Partition Tolerance
The system continues to operate
despite network partitions
CAP Theorem: You can pick two at most
  Consistency                        Consistency
Every read receives the most recent  Availability          Partition
write or an error                                          Tolerance
  Availability
Every read receives a (non-error)
response
  Partition Tolerance
The system continues to operate
despite network partitions
Different DBMS make different trade-offs
CA: Traditional                                                        CP: MongoDB
Single Node                                                            (When configured for
                                                                       strong consistency (e.g.,
Relational                       Consistency                           using a majority write
                                                                       concern). During a network
DBMS                                                                   partition, it will sacrifice
                                                                       availability in the minority
(e.g., MySQL, Oracle,                                                  partition to ensure the
PostgreSQL)                                                            remaining available nodes
                                                                       hold consistent data.
AP: Eventually- Availability     Partition
Consistent DBMS                  Tolerance                                                             26
Most NoSQL databases
(e.g., Apache Cassandra, Amazon
DynamoDB, CouchDB)
            CS 35L Software Construction: Lecture 8 ­ Data Management
            Tobias Dürschmid
SQL Is Optional in this course! If you like to use it
in your project, Use Remy's Tutorial
· https://remy.wang/cs143/notes/sql/sql.html
· I am also working on a SQL tutorial, but it's not ready yet
Please fill out your Exit Tickets on Bruin Learn!
Please summarize three database operations you learned today.
Please describe in your own words what the ACID properties are and why they are
important for data management.
Please leave any questions that you have about today's material and things that are
still unclear or confusing to you (if none, simply write N/A)
Credits: These slides use images from Flaticon.com (Creators: Freepik, phatplus)

```
