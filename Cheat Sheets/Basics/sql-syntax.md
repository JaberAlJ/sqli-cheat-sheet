# Common SQL Syntax — Quick Reference

This file provides a concise reference of the SQL syntax and constructs you should know when learning about SQL Injection. Focus is on *understanding normal SQL* so you can recognize how malformed input can change queries.

---

## 1. Identifiers & literals

* **Identifiers**: table and column names (e.g. `users`, `email`, `user_id`).
* **String literals**: enclosed in single quotes: `'hello'` (double quotes are used for identifiers in some DBMS).
* **Numeric literals**: e.g. `123`, `3.14` (no quotes).
* **NULL**: special marker for missing value: `NULL`.

**Note:** Different DBMS have small differences (e.g. MySQL accepts backticks `` `col` ``, PostgreSQL uses double quotes `"col"`).

---

## 2. Comments

* SQL single-line comment (most DBMS): `-- comment` (two hyphens to end of line).
* C-style comment: `/* comment */` (multi-line).

Comments are useful for truncating the rest of a query during testing — remember to use ethically and only in authorized environments.

---

## 3. Basic SELECT structure

```sql
SELECT column1, column2
FROM table_name
WHERE condition
ORDER BY column1 [ASC|DESC]
LIMIT n OFFSET m;  -- MySQL/Postgres/SQLite style
```

Example:

```sql
SELECT id, username FROM users WHERE id = 42;
```

---

## 4. Filtering & operators

* Comparison: `=`, `!=` or `<>`, `<`, `>`, `<=`, `>=`
* Logical: `AND`, `OR`, `NOT`
* Pattern matching: `LIKE 'abc%'`, `ILIKE` (Postgres case-insensitive)
* Range: `BETWEEN a AND b`
* Membership: `IN (1,2,3)`
* Null checks: `IS NULL`, `IS NOT NULL`

---

## 5. Joins (common forms)

```sql
-- Inner join
SELECT * FROM a INNER JOIN b ON a.id = b.a_id;

-- Left join (preserves left-side rows)
SELECT * FROM a LEFT JOIN b ON a.id = b.a_id;
```

---

## 6. Aggregation & grouping

```sql
SELECT role, COUNT(*) AS users_count
FROM users
GROUP BY role
HAVING COUNT(*) > 10;
```

---

## 7. Subqueries & UNION

```sql
-- Subquery in WHERE
SELECT * FROM products WHERE category_id = (SELECT id FROM categories WHERE name='Toys');

-- Combine results (columns must match)
SELECT username FROM users
UNION
SELECT name FROM admins;
```

---

## 8. Data modification basics

* `INSERT INTO table (col1, col2) VALUES (v1, v2);`
* `UPDATE table SET col = value WHERE condition;`
* `DELETE FROM table WHERE condition;`
* `TRUNCATE TABLE table;` (fast, often DDL-level)

---

## 9. Useful functions and casts

* String: `CONCAT()`, `SUBSTRING()`, `LENGTH()`/`LEN()`
* Numeric: `CAST(expr AS INT)`, arithmetic operators
* Date/time: `NOW()`, `CURRENT_TIMESTAMP`

DBMS-specific functions differ — check the examples files for MySQL/MSSQL/Oracle/Postgres variants.

---

## 10. Things to notice from a security perspective

* **Where user input is placed** (string vs numeric context) changes how it must be encoded/escaped.
* **Quotes and comment syntax** can be used (in tests) to alter query structure — understanding them is essential for prevention.
* **Multiple statements**: some DBMS allow `;` to chain statements (often restricted by application APIs).
* **DBMS-specific quirks** (quoting, concatenation, error messages) affect both attacks and defenses.