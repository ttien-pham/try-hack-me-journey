# SQL Fundamentals

> TryHackMe | Cyber Security 101  
> Room: [SQL Fundamentals](https://tryhackme.com/room/sqlfundamentals)  
> Difficulty: Easy | Tags: SQL, Database, Web Hacking

---

## 📌 Overview

Databases are so ubiquitous in computing that virtually every cybersecurity
role touches them. Whether you are attacking a web application via SQL
injection, working in a SOC querying a SIEM, configuring authentication
systems, or using threat detection tools — databases are always involved.
This room builds the complete foundation: from what databases are, to writing
queries that filter, sort, aggregate, and manipulate data.

**Topics covered:**
- Relational vs non-relational databases
- Tables, rows, columns, primary keys, foreign keys
- What SQL is and what a DBMS does
- Database and table management statements
- CRUD operations (INSERT, SELECT, UPDATE, DELETE)
- Clauses (DISTINCT, GROUP BY, ORDER BY, HAVING)
- Operators (LIKE, AND, OR, NOT, BETWEEN, comparison operators)
- Functions — string (CONCAT, GROUP_CONCAT, SUBSTRING, LENGTH) and
  aggregate (COUNT, SUM, MAX, MIN)

---

## 🔌 Setup — Connecting to MySQL

```bash
mysql -u root -p
# password: tryhackme
```

Expected output:
```
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.39-0ubuntu0.20.04.1 (Ubuntu)
mysql>
```

All SQL statements end with `;` — if nothing happens after pressing Enter,
you likely forgot the semicolon.

---

## 🧠 Task 2 — Databases 101

### What is a Database?

A database is an organised collection of structured information or data that
is easily accessible and can be manipulated or analysed. Real-world examples:
- **Authentication systems** — usernames and passwords stored and checked on login
- **Social media** — posts, comments, likes, watch history (Instagram, Netflix)
- **E-commerce** — product catalogues, orders, customer records

Databases are not just for large-scale companies. Any business that needs
to store and retrieve data will configure one.

---

### Relational vs Non-Relational

| Type | Structure | Best for | Example use case |
|------|-----------|---------|-----------------|
| **Relational** (SQL) | Fixed tables with defined columns and rows | Consistent, structured data where accuracy is critical | E-commerce transactions, banking |
| **Non-relational** (NoSQL) | Flexible — documents, key-value pairs, collections | Data that varies greatly in format | Social media user-generated content, IoT data |

**Non-relational example document:**
```json
{
    "_id": ObjectId("4556712cd2b2397ce1b47661"),
    "name": { "first": "Thomas", "last": "Anderson" },
    "date_of_birth": new Date('Sep 2, 1964'),
    "occupation": ["The One"],
    "steps_taken": NumberLong(4738947387743977493)
}
```

> 💡 The deciding factor is always context. If the incoming data format is
> predictable and accuracy is non-negotiable → relational. If the data is
> heterogeneous and flexibility or scale matters more → non-relational.

---

### Tables, Rows, and Columns

All data in a relational database lives in tables. When creating a table,
you define columns — each column has a name and a **data type** that enforces
what values are allowed. If a value doesn't match the type, it is rejected.

**Core data types:**

| Type | Description | Example |
|------|-------------|---------|
| `INT` | Integer (whole number) | `book_id INT` |
| `VARCHAR(n)` | Variable-length string, max n characters | `name VARCHAR(255)` |
| `DATE` | Date in YYYY-MM-DD format | `published_date DATE` |
| `FLOAT` / `DECIMAL` | Number with decimal point | `price DECIMAL(10,2)` |
| `BOOLEAN` | True or false | `in_stock BOOLEAN` |

**Visual model:**
```
Table: books
┌─────────┬────────────────────────────┬──────────────────┐
│ book_id │ book_name                  │ publication_date │  ← columns
├─────────┼────────────────────────────┼──────────────────┤
│ 1       │ Android Security Internals │ 2014-10-14       │  ← row (record)
│ 2       │ Bug Bounty Bootcamp        │ 2021-11-16       │  ← row (record)
└─────────┴────────────────────────────┴──────────────────┘
```

---

### Primary and Foreign Keys

| Key | Purpose | Rule | Example |
|-----|---------|------|---------|
| **Primary key** | Uniquely identifies each row within its table | Only ONE per table; value must be unique and not null | `book_id` — no two books share the same ID |
| **Foreign key** | Links a row in one table to a row in another | Can have MULTIPLE per table | `author_id` in the `books` table points to `id` in the `authors` table |

> 💡 Think of primary keys like matriculation numbers at a university — each
> student gets a unique number even if two students share the same name.
> Foreign keys are the bridge that makes a database "relational" — they allow
> you to query across multiple tables.

**Task answers:**
- Data that varies greatly in format → **Non-relational database**
- Data reliably in the same structured format → **Relational database**
- A record inserted into a table is represented as a → **row**
- Key that links one table to another → **foreign key**
- Key that ensures a record is unique within a table → **primary key**

---

## 🧠 Task 3 — SQL

### What is a DBMS?

A **Database Management System (DBMS)** is software that acts as an interface
between the end user and the database — it receives queries, executes them,
and returns results.

```
User / Application
       ↓  SQL query
     DBMS  (MySQL, PostgreSQL, Oracle, MongoDB, MariaDB...)
       ↓
    Database (actual data stored on disk)
```

Examples: MySQL, PostgreSQL, Oracle Database, MariaDB, MongoDB.

### What is SQL?

**SQL (Structured Query Language)** is used to query, define, and manipulate
data in a relational database. Benefits:

| Benefit | Why it matters |
|---------|---------------|
| **Fast** | Relational databases return massive datasets almost instantaneously |
| **Easy to learn** | Written in plain English — readable without deep programming knowledge |
| **Reliable** | Strict data typing rejects mismatched values — data integrity is enforced |
| **Flexible** | Complex queries, filtering, aggregation, and analysis all possible |

**Task answers:**
- Interface between database and end user → **DBMS**
- Query language used to interact with a relational database → **SQL**

---

## 🧠 Task 4 — Database and Table Statements

### Database Statements

```sql
-- Create a new database
CREATE DATABASE thm_bookmarket_db;

-- List all databases on the server
SHOW DATABASES;

-- Select a database to work with (makes it active for subsequent queries)
USE thm_bookmarket_db;

-- Permanently delete a database — irreversible
DROP DATABASE thm_bookmarket_db;
```

### Table Statements

```sql
-- Create a table with typed columns
CREATE TABLE book_inventory (
    book_id          INT AUTO_INCREMENT PRIMARY KEY,
    book_name        VARCHAR(255) NOT NULL,
    publication_date DATE
);
```

Breaking down this example:
- `book_id INT AUTO_INCREMENT PRIMARY KEY` — integer, auto-increments (1, 2, 3...), uniquely identifies each record
- `book_name VARCHAR(255) NOT NULL` — variable text up to 255 chars, cannot be empty
- `publication_date DATE` — stores a date value

```sql
-- List all tables in the active database
SHOW TABLES;

-- Show the structure of a table (columns, types, constraints)
DESCRIBE book_inventory;
-- Can also be written as: DESC book_inventory;
```

`DESCRIBE` output example:
```
+------------------+--------------+------+-----+---------+----------------+
| Field            | Type         | Null | Key | Default | Extra          |
+------------------+--------------+------+-----+---------+----------------+
| book_id          | int          | NO   | PRI | NULL    | auto_increment |
| book_name        | varchar(255) | NO   |     | NULL    |                |
| publication_date | date         | YES  |     | NULL    |                |
+------------------+--------------+------+-----+---------+----------------+
```

```sql
-- Modify an existing table: add a column
ALTER TABLE book_inventory
ADD page_count INT;

-- ALTER can also: rename columns, change data types, remove columns
-- Example: remove a column
ALTER TABLE book_inventory
DROP COLUMN page_count;

-- Permanently delete a table — irreversible
DROP TABLE book_inventory;
```

> ⚠️ `DROP` is permanent — there is no undo. Always double-check before
> dropping anything in a production environment.

**Task answers:**
- `SHOW DATABASES;` reveals the flag database:
  **`THM{575a947132312f97b30ee5aeebba629b723d30f9}`**
- `USE task_4_db; SHOW TABLES;` reveals:
  **`THM{692aa7eaec2a2a827f4d1a8bed1f90e5e49d2410}`**

---

## 🧠 Task 5 — CRUD Operations

CRUD = **C**reate, **R**ead, **U**pdate, **D**elete — the four fundamental
data operations in any system that manages data.

We use the `thm_books` database for this task: `USE thm_books;`

---

### CREATE — INSERT

```sql
-- Insert a single record
INSERT INTO books (id, name, published_date, description)
VALUES (1, 'Android Security Internals', '2014-10-14',
        'An In-Depth Guide to Android\'s Security Architecture');
```

Output:
```
Query OK, 1 row affected (0.01 sec)
```

---

### READ — SELECT

```sql
-- Select ALL columns and ALL rows
SELECT * FROM books;
```

Output:
```
+----+----------------------------+----------------+------------------------------------------------------+
| id | name                       | published_date | description                                          |
+----+----------------------------+----------------+------------------------------------------------------+
|  1 | Android Security Internals | 2014-10-14     | An In-Depth Guide to Android's Security Architecture |
+----+----------------------------+----------------+------------------------------------------------------+
```

```sql
-- Select specific columns only
SELECT name, description FROM books;
```

Output:
```
+----------------------------+------------------------------------------------------+
| name                       | description                                          |
+----------------------------+------------------------------------------------------+
| Android Security Internals | An In-Depth Guide to Android's Security Architecture |
+----------------------------+------------------------------------------------------+
```

---

### UPDATE

```sql
-- Update a specific record — WHERE is critical
UPDATE books
SET description = 'An In-Depth Guide to Android\'s Security Architecture.'
WHERE id = 1;
```

Output:
```
Query OK, 1 row affected (0.00 sec)
Rows matched: 1  Changed: 1  Warnings: 0
```

> ⚠️ `UPDATE` **without a `WHERE` clause** modifies every single row in the
> table. Always run a `SELECT` with the same `WHERE` condition first to
> confirm you are targeting the right rows.

---

### DELETE

```sql
-- Delete a specific record
DELETE FROM books WHERE id = 1;
```

Output:
```
Query OK, 1 row affected (0.00 sec)
```

> ⚠️ Same warning as `UPDATE` — `DELETE` without `WHERE` removes all rows
> from the table (the table itself remains, but all data is gone).

---

### CRUD Summary

| Operation | Statement | What it does |
|-----------|-----------|-------------|
| **Create** | `INSERT INTO` | Adds a new record to the table |
| **Read** | `SELECT` | Retrieves records from the table |
| **Update** | `UPDATE ... SET` | Modifies existing data in the table |
| **Delete** | `DELETE FROM` | Removes records from the table |

### Practical queries on `tools_db`

```sql
USE tools_db;

-- Find tool used for man-in-the-middle attacks on wireless networks
SELECT * FROM hacking_tools
WHERE description LIKE '%man-in-the-middle%';
-- Answer: Wi-Fi Pineapple

-- Find shared category of USB Rubber Ducky and Bash Bunny
SELECT name, category FROM hacking_tools
WHERE name IN ('USB Rubber Ducky', 'Bash Bunny');
-- Answer: USB attacks
```

**Task answers:**
- Tool for MITM on wireless networks → **Wi-Fi Pineapple**
- Shared category of USB Rubber Ducky and Bash Bunny → **USB attacks**

---

## 🧠 Task 6 — Clauses

A clause is a part of a SQL statement that specifies criteria for the data
being retrieved or manipulated. We already know `FROM` (specifies the table)
and `WHERE` (filters rows). This task covers four more.

We use `thm_books` database: `USE thm_books;`

---

### DISTINCT

Eliminates duplicate values — returns only unique results.

```sql
-- Without DISTINCT — Ethical Hacking appears twice
SELECT * FROM books;
-- 6 rows returned (Ethical Hacking duplicated)

-- With DISTINCT — each name appears once
SELECT DISTINCT name FROM books;
```

Output:
```
+----------------------------+
| name                       |
+----------------------------+
| Android Security Internals |
| Bug Bounty Bootcamp        |
| Car Hacker's Handbook      |
| Designing Secure Software  |
| Ethical Hacking            |
+----------------------------+
5 rows in set (0.00 sec)
```

---

### GROUP BY

Aggregates data from multiple records and groups results by a column.
Typically used with aggregate functions like `COUNT()`.

```sql
SELECT name, COUNT(*)
FROM books
GROUP BY name;
```

Output:
```
+----------------------------+----------+
| name                       | COUNT(*) |
+----------------------------+----------+
| Android Security Internals |        1 |
| Bug Bounty Bootcamp        |        1 |
| Car Hacker's Handbook      |        1 |
| Designing Secure Software  |        1 |
| Ethical Hacking            |        2 |
+----------------------------+----------+
```

---

### ORDER BY

Sorts query results in ascending (`ASC`) or descending (`DESC`) order.

```sql
-- Ascending by date (oldest first)
SELECT * FROM books ORDER BY published_date ASC;

-- Descending by date (newest first)
SELECT * FROM books ORDER BY published_date DESC;
```

Ascending output (first and last rows):
```
|  1 | Android Security Internals | 2014-10-14 | ...  ← oldest
...
|  4 | Designing Secure Software  | 2021-12-21 | ...  ← newest
```

---

### HAVING

Filters groups or aggregated results — it runs *after* `GROUP BY`, unlike
`WHERE` which runs *before* grouping.

```sql
SELECT name, COUNT(*)
FROM books
GROUP BY name
HAVING name LIKE '%Hack%';
```

Output:
```
+-----------------------+----------+
| name                  | COUNT(*) |
+-----------------------+----------+
| Car Hacker's Handbook |        1 |
| Ethical Hacking       |        2 |
+-----------------------+----------+
```

> 💡 **WHERE vs HAVING — the key difference:**  
> `WHERE` filters individual rows *before* grouping.  
> `HAVING` filters groups *after* `GROUP BY` has run.  
> Rule of thumb: if you need to filter on an aggregate value (`COUNT > 2`,
> `SUM > 100`), that goes in `HAVING`, not `WHERE`.

### Practical queries on `tools_db`

```sql
USE tools_db;

-- Count distinct categories
SELECT COUNT(DISTINCT category) FROM hacking_tools;
-- Answer: 6

-- First tool in ascending order
SELECT name FROM hacking_tools ORDER BY name ASC LIMIT 1;
-- Answer: Bash Bunny

-- First tool in descending order
SELECT name FROM hacking_tools ORDER BY name DESC LIMIT 1;
-- Answer: Wi-Fi Pineapple
```

**Task answers:**
- Total distinct categories → **6**
- First tool ascending → **Bash Bunny**
- First tool descending → **Wi-Fi Pineapple**

---

## 🧠 Task 7 — Operators

Operators let you build logic into queries. We use `thm_books2` database:
`USE thm_books2;`

---

### Logical Operators

#### LIKE — Pattern Matching

```sql
-- description contains the word "guide" anywhere
SELECT * FROM books WHERE description LIKE '%guide%';
```

Output:
```
|  1 | Android Security Internals | ... | An In-Depth Guide to Android's Security Architecture   |
|  2 | Bug Bounty Bootcamp        | ... | The Guide to Finding and Reporting Web Vulnerabilities |
|  3 | Car Hacker's Handbook      | ... | A Guide for the Penetration Tester                     |
|  4 | Designing Secure Software  | ... | A Guide for Developers                                 |
```

Wildcards:
- `%` — any number of characters (including zero)
- `_` — exactly one character

---

#### AND — Both conditions must be true

```sql
SELECT * FROM books
WHERE category = 'Offensive Security' AND name = 'Bug Bounty Bootcamp';
```

Output:
```
|  2 | Bug Bounty Bootcamp | 2021-11-16 | The Guide to Finding and Reporting Web Vulnerabilities | Offensive Security |
```

---

#### OR — At least one condition must be true

```sql
SELECT * FROM books
WHERE name LIKE '%Android%' OR name LIKE '%iOS%';
```

Output:
```
|  1 | Android Security Internals | 2014-10-14 | An In-Depth Guide to Android's Security Architecture | Defensive Security |
```

---

#### NOT — Reverses a condition

```sql
-- Books whose description does NOT contain "guide"
SELECT * FROM books
WHERE NOT description LIKE '%guide%';
```

Output:
```
|  5 | Ethical Hacking | 2021-11-02 | A Hands-on Introduction to Breaking In | Offensive Security |
```

---

#### BETWEEN — Tests if a value is within a range (inclusive)

```sql
-- Books with id between 2 and 4
SELECT * FROM books WHERE id BETWEEN 2 AND 4;
```

Output:
```
|  2 | Bug Bounty Bootcamp       | ...
|  3 | Car Hacker's Handbook     | ...
|  4 | Designing Secure Software | ...
```

---

### Comparison Operators

| Operator | Meaning | Example |
|----------|---------|---------|
| `=` | Equal to | `WHERE name = 'Designing Secure Software'` |
| `!=` or `<>` | Not equal to | `WHERE category != 'Offensive Security'` |
| `<` | Less than | `WHERE published_date < '2020-01-01'` |
| `>` | Greater than | `WHERE published_date > '2020-01-01'` |
| `<=` | Less than or equal to | `WHERE published_date <= '2021-11-15'` |
| `>=` | Greater than or equal to | `WHERE published_date >= '2021-11-02'` |

```sql
-- Equal: exact match
SELECT * FROM books WHERE name = 'Designing Secure Software';

-- Not equal: exclude a category
SELECT * FROM books WHERE category != 'Offensive Security';
-- Returns: Android Security Internals, Designing Secure Software (both Defensive)

-- Less than: published before 2020
SELECT * FROM books WHERE published_date < '2020-01-01';
-- Returns: Android Security Internals (2014), Car Hacker's Handbook (2016)

-- Greater than: published after 2020
SELECT * FROM books WHERE published_date > '2020-01-01';
-- Returns: Bug Bounty Bootcamp, Designing Secure Software, Ethical Hacking

-- Less than or equal: on or before Nov 15, 2021
SELECT * FROM books WHERE published_date <= '2021-11-15';
-- Returns: Android Security Internals, Car Hacker's Handbook, Ethical Hacking

-- Greater than or equal: on or after Nov 2, 2021
SELECT * FROM books WHERE published_date >= '2021-11-02';
-- Returns: Bug Bounty Bootcamp, Designing Secure Software, Ethical Hacking
```

### Practical queries on `tools_db`

```sql
USE tools_db;

-- Multi-tool useful for pentesters and geeks
SELECT * FROM hacking_tools
WHERE category = 'Multi-tool' AND description LIKE '%pentesters%';
-- Answer: Flipper Zero

-- Category of tools with amount >= 300
SELECT category FROM hacking_tools WHERE amount >= 300;
-- Answer: RFID cloning

-- Network intelligence tool with amount < 100
SELECT * FROM hacking_tools
WHERE category = 'Network intelligence' AND amount < 100;
-- Answer: Lan Turtle
```

**Task answers:**
- Multi-tool for pentesters and geeks → **Flipper Zero**
- Category of tools with amount >= 300 → **RFID cloning**
- Network intelligence tool with amount < 100 → **Lan Turtle**

---

## 🧠 Task 8 — Functions

Functions process data and return a result. Two categories: string functions
and aggregate functions.

---

### String Functions

#### CONCAT() — Combine multiple strings into one

```sql
SELECT CONCAT(name, ' is a type of ', category, ' book.') AS book_info
FROM books;
```

Output:
```
+------------------------------------------------------------------+
| book_info                                                        |
+------------------------------------------------------------------+
| Android Security Internals is a type of Defensive Security book. |
| Bug Bounty Bootcamp is a type of Offensive Security book.        |
| Car Hacker's Handbook is a type of Offensive Security book.      |
| Designing Secure Software is a type of Defensive Security book.  |
| Ethical Hacking is a type of Offensive Security book.            |
+------------------------------------------------------------------+
```

---

#### GROUP_CONCAT() — Concatenate values from multiple rows into one field

```sql
SELECT category, GROUP_CONCAT(name SEPARATOR ', ') AS books
FROM books
GROUP BY category;
```

Output:
```
+--------------------+-------------------------------------------------------------+
| category           | books                                                       |
+--------------------+-------------------------------------------------------------+
| Defensive Security | Android Security Internals, Designing Secure Software       |
| Offensive Security | Bug Bounty Bootcamp, Car Hacker's Handbook, Ethical Hacking |
+--------------------+-------------------------------------------------------------+
```

---

#### SUBSTRING() — Extract part of a string

Syntax: `SUBSTRING(column, start_position, length)`

```sql
-- Extract only the year (first 4 characters) from published_date
SELECT SUBSTRING(published_date, 1, 4) AS published_year FROM books;
```

Output:
```
+----------------+
| published_year |
+----------------+
| 2014           |
| 2021           |
| 2016           |
| 2021           |
| 2021           |
+----------------+
```

---

#### LENGTH() — Count characters in a string

Includes spaces and punctuation.

```sql
SELECT LENGTH(name) AS name_length FROM books;
```

Output:
```
+-------------+
| name_length |
+-------------+
|          26 |  ← Android Security Internals
|          19 |  ← Bug Bounty Bootcamp
|          21 |  ← Car Hacker's Handbook
|          25 |  ← Designing Secure Software
|          15 |  ← Ethical Hacking
+-------------+
```

---

### Aggregate Functions

Aggregate functions operate on a set of rows and return a single summary value.

#### COUNT() — Count the number of rows

```sql
SELECT COUNT(*) AS total_books FROM books;
```

Output:
```
+-------------+
| total_books |
+-------------+
|           5 |
+-------------+
```

---

#### SUM() — Total sum of a numeric column

```sql
SELECT SUM(price) AS total_price FROM books;
```

Output:
```
+-------------+
| total_price |
+-------------+
|      249.95 |
+-------------+
```

---

#### MAX() — Highest value in a column

```sql
SELECT MAX(published_date) AS latest_book FROM books;
```

Output:
```
+-------------+
| latest_book |
+-------------+
| 2021-12-21  |
+-------------+
```

---

#### MIN() — Lowest value in a column

```sql
SELECT MIN(published_date) AS earliest_book FROM books;
```

Output:
```
+---------------+
| earliest_book |
+---------------+
| 2014-10-14    |
+---------------+
```

---

### Practical queries on `tools_db`

```sql
USE tools_db;

-- Tool with longest name by character count
SELECT name FROM hacking_tools ORDER BY LENGTH(name) DESC LIMIT 1;
-- Answer: USB Rubber Ducky

-- Total sum of all tool prices
SELECT SUM(amount) FROM hacking_tools;
-- Answer: 1444

-- Tool names where amount does not end in 0, grouped with " & "
SELECT GROUP_CONCAT(name SEPARATOR ' & ') AS tools
FROM hacking_tools
WHERE amount % 10 != 0;
-- Answer: Flipper Zero & iCopy-XS
```

**Task answers:**
- Tool with longest name → **USB Rubber Ducky**
- Total sum of all tools → **1444**
- Tools where amount doesn't end in 0 (concatenated) → **Flipper Zero & iCopy-XS**

---

## 📋 Full SQL Quick Reference

### Database & Table Management

```sql
-- Databases
SHOW DATABASES;
CREATE DATABASE db_name;
USE db_name;
DROP DATABASE db_name;

-- Tables
SHOW TABLES;
DESCRIBE table_name;          -- or: DESC table_name
CREATE TABLE table_name (
    col1 INT AUTO_INCREMENT PRIMARY KEY,
    col2 VARCHAR(255) NOT NULL,
    col3 DATE
);
ALTER TABLE table_name ADD COLUMN col TYPE;
ALTER TABLE table_name DROP COLUMN col;
DROP TABLE table_name;
```

### CRUD

```sql
INSERT INTO table (col1, col2) VALUES (val1, val2);
SELECT col1, col2 FROM table WHERE condition;
SELECT * FROM table;
UPDATE table SET col = val WHERE condition;
DELETE FROM table WHERE condition;
```

### Clauses

```sql
SELECT DISTINCT col FROM table;
SELECT * FROM table ORDER BY col ASC;
SELECT * FROM table ORDER BY col DESC;
SELECT col, COUNT(*) FROM table GROUP BY col;
SELECT col, COUNT(*) FROM table GROUP BY col HAVING condition;
SELECT * FROM table LIMIT n;
```

### Operators

```sql
WHERE col LIKE '%pattern%'        -- contains pattern
WHERE col LIKE 'prefix%'          -- starts with
WHERE col LIKE '%suffix'          -- ends with
WHERE col BETWEEN val1 AND val2   -- inclusive range
WHERE col IN ('a', 'b', 'c')      -- matches any listed value
WHERE col1 = val AND col2 != val  -- both true
WHERE col1 = val OR col2 = val    -- either true
WHERE NOT col LIKE '%pattern%'    -- excludes pattern
WHERE col > val
WHERE col >= val
WHERE col < val
WHERE col <= val
```

### Functions

```sql
-- String
CONCAT(col1, ' ', col2)
GROUP_CONCAT(col SEPARATOR ', ')
SUBSTRING(col, start, length)
LENGTH(col)
UPPER(col)
LOWER(col)

-- Aggregate
COUNT(*)
COUNT(DISTINCT col)
SUM(col)
AVG(col)
MAX(col)
MIN(col)
```

---

## 💡 Insights & Things to Remember

**SQL is both offensive and defensive:**  
Writing SQL queries is the same foundation needed to understand SQL injection.
When you see `WHERE username = '$input'` in application code, you know exactly
what an attacker can inject and why — because you know how the query works.

**Always test your WHERE with SELECT before UPDATE or DELETE:**  
Run `SELECT * FROM table WHERE condition` first. Confirm you are targeting
exactly the right rows. Only then run the `UPDATE` or `DELETE`. This habit
prevents destructive, irreversible mistakes.

**WHERE vs HAVING — know when each runs:**  
`WHERE` filters rows *before* grouping. `HAVING` filters groups *after*
aggregation. If you need to filter on `COUNT(*)` or `SUM()`, it goes in
`HAVING`. Trying to use `WHERE COUNT(*) > 2` will give you an error.

**NULL is not zero and not empty string:**  
`WHERE col = NULL` returns nothing — always. The correct syntax is
`WHERE col IS NULL`. Forgetting this causes invisible data loss in queries.

**AUTO_INCREMENT handles ID assignment for you:**  
You don't need to track the next ID manually. Define `book_id INT
AUTO_INCREMENT PRIMARY KEY` and the database assigns 1, 2, 3... automatically
on each insert.

---

## ⚖️ Reflection

**What clicked:**
- The relational vs non-relational distinction becomes intuitive through use
  cases — e-commerce needs relational (ACID, consistency), social media
  content needs non-relational (scale, schema flexibility)
- `HAVING` vs `WHERE` only makes sense once you understand query execution
  order: `WHERE` runs first (on individual rows), then `GROUP BY`, then
  `HAVING` (on groups)
- `GROUP_CONCAT` is surprisingly powerful for formatting results during
  investigations — pulling comma-separated lists from multiple rows into one
  field saves a lot of manual work

**Still unclear:**
- **JOINs** — this room intentionally skips them, but relational databases
  without JOINs feel incomplete. `INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN` are
  the next thing to learn.
- **Indexes** — how does MySQL speed up `WHERE` queries on tables with
  millions of rows? Understanding indexes is important before working with
  production databases.
- **Transactions** — `BEGIN`, `COMMIT`, `ROLLBACK` are critical for
  understanding how SQL injection can modify or delete data atomically.

**Next:**
- SQL Injection room on TryHackMe — apply these fundamentals offensively
- Practice `JOIN` queries to query across multiple related tables
- Explore `EXPLAIN` to understand how MySQL executes a query internally

---

## 📚 References

- [TryHackMe - SQL Fundamentals](https://tryhackme.com/room/sqlfundamentals)
- [MySQL 8.0 Reference Manual](https://dev.mysql.com/doc/refman/8.0/en/)
- [W3Schools SQL Reference](https://www.w3schools.com/sql/)
- [PortSwigger SQL Injection](https://portswigger.net/web-security/sql-injection) — next step
