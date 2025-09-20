# SQL Statements — SELECT, INSERT, UPDATE, DELETE (Quick Reference)

This page covers the common Data Query and Manipulation Language (DQL/DML) statements you’ll encounter when learning about SQL Injection. Understanding these statements — and where user input commonly appears — is essential for both attacking (in lab) and defending applications.

---

## 1. SELECT — reading data

**Purpose:** Retrieve rows/columns from one or more tables.

Basic form:

```sql
SELECT column1, column2
FROM table_name
WHERE condition
ORDER BY column1 [ASC|DESC]
LIMIT n OFFSET m;
```

Example:

```sql
SELECT id, username, email
FROM users
WHERE username = 'alice';
```

Common injection vectors:

* Filtering values in `WHERE` (e.g., `username = '<input>'`).
* ORDER BY and LIMIT when user controls column or number.
* UNION SELECT to combine attacker-controlled result sets (requires matching column counts/types).

Example UNION attack pattern:

```sql
-- Original query appends user input inside WHERE
... WHERE id = 'USER_INPUT'
-- Attacker supplies: ' UNION SELECT null, version(), user() --
```

Security note: Avoid concatenating raw user input into queries. Use parameterized queries / prepared statements.

---

## 2. INSERT — adding rows

**Purpose:** Add new rows to a table.

Basic form:

```sql
INSERT INTO table_name (col1, col2)
VALUES (val1, val2);
```

Example:

```sql
INSERT INTO users (username, email, created_at)
VALUES ('bob', 'bob@example.com', NOW());
```

Common injection vectors:

* Values inserted directly from user input (e.g., sign-up forms).
* Some applications build dynamic `INSERT` statements — risky when not parameterized.

Security note: Use typed parameters and validate/escape input. Be careful with attacker-supplied column names or bulk `INSERT` constructs.

---

## 3. UPDATE — modifying rows

**Purpose:** Change existing rows.

Basic form:

```sql
UPDATE table_name
SET col1 = value1, col2 = value2
WHERE condition;
```

Example:

```sql
UPDATE users
SET email = 'new@example.com'
WHERE id = 42;
```

Common injection vectors:

* `WHERE` clause injection can escalate scope (e.g., changing many rows if condition is bypassed).
* Unescaped values in `SET` can break query structure.

Dangerous example (unsafe concatenation):

```sql
-- Application does: "UPDATE accounts SET balance = " + userInput + " WHERE id=..."
-- Attacker supplies: "0; DROP TABLE accounts; --"
```

Many DB APIs disallow multiple statements, but some do allow them — never rely on that for safety.

Security note: Principle of least privilege — DB accounts used by the app should not have unnecessary write/delete rights. Use prepared statements and strict input validation.

---

## 4. DELETE — removing rows

**Purpose:** Remove rows from a table.

Basic form:

```sql
DELETE FROM table_name WHERE condition;
```

Example:

```sql
DELETE FROM sessions WHERE last_activity < NOW() - INTERVAL '30 days';
```

Common injection vectors:

* Missing or manipulated `WHERE` clause can lead to mass deletion (`DELETE FROM table_name;`).
* User-controlled conditions are high-risk.

Security note: Require safe authorization checks before deletion; consider using soft-delete (`is_deleted` flag) rather than hard deletes where appropriate.

---

## 5. Best practices to prevent SQL Injection (applies to all statements)

1. **Use parameterized queries / prepared statements** — never build SQL by concatenating raw user input.

   * Example (pseudo): `db.query('SELECT * FROM users WHERE id = ?', [userId])`.
2. **Validate and sanitize input** — enforce allowed characters, length, and type checks.
3. **Least privilege** — run the application with a DB account that has only necessary permissions.
4. **Use ORM / query builders** carefully — they help but are not a silver bullet; understand how they construct SQL.
5. **Escape identifiers correctly** if user input ever controls table/column names — better: disallow it.
6. **Use stored procedures cautiously** — they can reduce some risks but can still be vulnerable if they concatenate inputs.
7. **Logging & monitoring** — detect anomalies like unexpected query shapes or high error rates.

---

## 6. Quick examples: safe vs unsafe

**Unsafe (concatenation):**

```python
# Python pseudo-code
query = "SELECT * FROM users WHERE username = '" + username + "'"
cursor.execute(query)
```

**Safe (parameterized):**

```python
query = "SELECT * FROM users WHERE username = %s"
cursor.execute(query, (username,))
```

---

## 7. Where to practice safely

> [!TIP]
> Use intentionally vulnerable labs (e.g., OWASP Juice Shop, DVWA, PortSwigger Labs) or local DB instances.

> [!CAUTION]
> Never test on production or systems without explicit written permission.