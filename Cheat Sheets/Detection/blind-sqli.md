# Blind SQL Injection — Detection & Techniques

> [!WARNING]
> Blind SQLi techniques are powerful and noisy. Use them **only** in authorized testing environments (local labs, intentionally vulnerable apps, or with explicit written permission). Do not test systems you do not own.

---

## What is Blind SQL Injection?

Blind SQLi occurs when an application does not return query results or helpful error messages, but **behaves differently** depending on whether a crafted SQL condition is true or false. Attackers infer database information by observing these behavioral differences.

Two main categories:

* **Boolean-based (content-based)** — infer data by comparing application responses for true/false conditions.
* **Time-based** — infer data by causing deliberate delays in the database and measuring response times.

---

## Why blind SQLi matters

* Many applications suppress errors and hide query output, making classic UNION or error-based attacks ineffective.
* Blind techniques allow data extraction even when responses appear identical to normal users.
* Extraction is slow (often character-by-character), which increases detection risk — and makes defensive monitoring effective.

---

## Boolean-based blind SQLi — approach & examples

**Approach:** Send two requests that differ only by a boolean condition. If responses differ, the condition evaluated differently in the DB.

**Proof-of-concept payloads** (commonly within a quoted parameter):

```sql
' AND 1=1 --
' AND 1=2 --
```

**Character extraction pattern:**

```sql
' AND SUBSTRING((SELECT user()),1,1) = 'a' --
```

Loop over characters and positions to reconstruct values.

**MySQL example (character test):**

```sql
' AND SUBSTRING((SELECT database()),1,1) = 'd' --
```

**Postgres example (character test):**

```sql
' AND SUBSTRING((SELECT current_database()),1,1) = 'd' --
```

**MSSQL example (character test):**

```sql
' AND SUBSTRING((SELECT SYSTEM_USER),1,1) = 'd' --
```

**Detection tips:**

* Measure response length, specific DOM elements, or HTTP headers for differences.
* Automate diffs: capture baseline response and compare byte-counts or DOM trees.
* Try simple true/false pairs first (`AND 1=1`, `AND 1=2`) to confirm influence.

**Caveats:**

* Some applications normalize responses making content differences tiny — use precise diffing tools.
* Character-by-character extraction is slow and easily detected.

---

## Time-based blind SQLi — approach & examples

**Approach:** Inject SQL that causes the DB to pause execution when a condition is true (e.g., `SLEEP()`), then measure response time to infer truth.

**MySQL timing payloads:**

```sql
' OR IF(SUBSTRING((SELECT user()),1,1)='a', SLEEP(5), 0) --
```

**Postgres timing payloads:**

```sql
' OR CASE WHEN SUBSTRING((SELECT current_database()),1,1)='d' THEN pg_sleep(5) ELSE pg_sleep(0) END --
```

**MSSQL timing payloads (using WAITFOR):**

```sql
' IF SUBSTRING((SELECT SYSTEM_USER),1,1)='d' WAITFOR DELAY '00:00:05' --
```

**Detection tips:**

* Send baseline request(s) to measure normal latency.
* Average multiple measurements to account for network jitter.
* Use delays significantly larger than normal variance (e.g., 3–7s) but be mindful of application timeouts.
* Time-based extraction is often slower than boolean-based but works when content differences are fully suppressed.

**Caveats:**

* Network instability can cause false positives/negatives; always repeat tests.
* Application/load balancers might cache or queue requests, affecting timing accuracy.

---

## Practical extraction strategy (high level)

1. Confirm injection point with simple `AND 1=1 / AND 1=2` or short sleep tests.
2. Determine DBMS (use fingerprinting payloads or prior error-based findings).
3. Use boolean extraction if responses differ in content; use time-based if they don’t.
4. Optimize character set and search order (e.g., test `[0-9a-z@_.-]` first) to reduce requests.
5. Rate-limit and throttle to reduce detection and effort.

---

## Defensive measures specific to blind attacks

* **Parameterized queries** eliminate injection at the source.
* **Consistent responses**: avoid exposing boolean-level differences; render generic pages for errors and missing data.
* **Function restrictions**: restrict or disable `SLEEP()`/`pg_sleep()`/`WAITFOR` for application DB users where feasible.
* **Monitoring & alerting**: detect many similar requests differing only by small payload changes or exhibiting repeated timing patterns.
* **Rate-limiting**: slow down high-frequency probes to make extraction impractically slow.

---

## Automation & tooling (authorized use)

* **Burp Suite**: use Intruder and Repeater for manual blind testing; use Comparer and logging for diffs.
* **sqlmap**: supports blind techniques (`--technique=B,T`), but be cautious — it’s noisy and can cause damage.
* **Custom scripts**: Python + `requests` or `https` for controlled, repeatable tests with statistical averaging for timings.

---

## Lab practice recommendations

* Use labs like PortSwigger Academy, OWASP Juice Shop, or deliberately vulnerable VMs.
* Start with boolean tests and progress to short, controlled extraction tasks.
* Keep experiments documented and within authorized scope.