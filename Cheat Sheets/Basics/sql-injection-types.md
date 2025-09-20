# SQL Injection Types — Overview, Examples, and Defensive Notes

> [!WARNING]
> **Ethics Reminder**
> The examples below are for education, defense, and authorized testing **only**. Do not use them against systems you do not own or do not have explicit written permission to test. Use intentionally vulnerable labs (PortSwigger Academy, OWASP Juice Shop, DVWA) or local environments.

---

## Contents

1. Union-based SQL Injection
2. Error-based SQL Injection
3. Boolean-based (Blind) SQL Injection
4. Time-based (Blind) SQL Injection
5. Quick mitigation checklist

---

## 1. Union-based SQL Injection

**What it is:** Attacker uses `UNION SELECT` to append a crafted result set to the application's original query, causing the application to return attacker-controlled data.

**Typical vulnerable pattern:**

```sql
-- Application builds query using untrusted input (simplified)
SELECT id, name FROM products WHERE id = 'USER_INPUT';
```

**Illustrative payload pattern:**

```
' UNION SELECT null, version() --
```

This payload attempts to close the original quoted value, union another SELECT, and comment out the rest. Real payloads must match the original query's column count and types.

**What to watch for:** Error messages or returned pages showing unexpected data (e.g., DB version, user names) after injection.

**Mitigation:**

* Use parameterized queries / prepared statements.
* Enforce strict type checks (e.g., numeric vs string).
* Do not reveal database errors to users.
* Apply least privilege so leaked data is limited.

---

## 2. Error-based SQL Injection

**What it is:** The attacker crafts input that causes the database to produce an error message which leaks information (e.g., full SQL, column names, DBMS version).

**Typical vulnerable pattern:** Applications that display raw DB errors back to the user.

**Illustrative payload pattern:**

```
' AND (SELECT 1 FROM (SELECT COUNT(*), CONCAT((SELECT database()), FLOOR(RAND(0)*2)) x FROM information_schema.tables GROUP BY x) t) --
```

(This demonstrates the idea of forcing a DB error that contains data; real syntax varies by DBMS.)

**What to watch for:** Detailed SQL or DB error output in responses, stack traces, or logs.

**Mitigation:**

* Do not expose database error messages to end users. Log them server-side instead.
* Parameterize queries and validate input.
* Harden DBMS configuration to minimize information exposed in errors.

---

## 3. Boolean-based (Blind) SQL Injection

**What it is:** The application does not return useful DB error messages or query results, but responds differently for true/false conditions (e.g., different page content or HTTP status). An attacker infers data one boolean at a time.

**Typical vulnerable pattern:** Queries that incorporate user input in conditional expressions; application responses differ subtly.

**Illustrative payload pattern:**

```
' AND (SELECT CASE WHEN (SUBSTRING(user(),1,1)='a') THEN 1 ELSE 0 END) = 1 --
```

The attacker changes the condition to test different characters of a value and observes the site's behavior.

**What to watch for:** Slight differences in response content, length, or timing between crafted true/false payloads.

**Mitigation:**

* Parameterize queries; avoid direct concatenation.
* Normalize and sanitize output so logic-level differences aren’t observable.
* Implement rate-limiting and anomaly detection to spot probing.

---

## 4. Time-based (Blind) SQL Injection

**What it is:** Similar to Boolean-based blind SQLi, but uses time delays (e.g., `SLEEP()` or `pg_sleep()`) to infer true/false by measuring response time.

**Typical vulnerable pattern:** User input is executed in a context where DB functions can introduce delays.

**Illustrative payload pattern:**

```
' OR (SELECT CASE WHEN (SUBSTRING(version(),1,1)='5') THEN SLEEP(5) ELSE 0 END) --
```

If the response is delayed by \~5 seconds, the tested condition is likely true.

**What to watch for:** Noticeable, reproducible increases in response time after sending crafted requests.

**Mitigation:**

* Same as boolean blind: parameterize queries, sanitize input.
* Limit or disable expensive DB functions for application accounts.
* Monitor for unusual latency patterns and probing behavior.

---

## 5. Quick mitigation checklist

* Use prepared statements / parameterization queries for all DB access.
* Validate and canonicalize user input (whitelist where possible).
* Escaping is brittle — prefer parameterization.
* Use least-privilege DB accounts; split read/write roles appropriately.
* Hide detailed DB errors from users; log them server-side.
* Implement Web Application Firewalls (WAFs) as an extra layer (but don’t rely on them solely).
* Monitor logs for abnormal queries and rate-limit suspicious endpoints.

---

## Further learning & safe practice

* Practice on labs: PortSwigger Academy, OWASP Juice Shop, DVWA, bWAPP.
* Read defensive guidance: OWASP SQL Injection Prevention Cheat Sheet.
* When testing, always obtain explicit permission and stay within scope.