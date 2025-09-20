# Detecting SQL Injection via Response Analysis

This page explains how to detect SQL Injection by observing changes in application responses — without relying on explicit error messages. Response analysis is central to blind SQLi detection (boolean- and time-based) and subtle fingerprinting of vulnerable endpoints.

> [!WARNING]
> Use these techniques only in authorized testing environments (labs, bug bounties with scope, or with written permission). Do not test systems you do not have explicit consent to probe.

---

## Why response analysis?

Some applications suppress errors and do not return raw SQL results. However, they often behave differently when fed crafted input. By carefully comparing responses we can infer whether input affects query logic.

Response changes to observe:

* HTTP status codes (200 vs 500)
* Response length / body differences
* HTML content changes or missing elements
* Redirects or HTTP headers differences
* Timing/latency differences

---

## 1. Boolean-based (Blind) detection

**Idea:** Inject payloads that evaluate to `TRUE` or `FALSE` and observe the application’s different responses.

Example payloads (commonly appended inside a quoted parameter):

```sql
' AND 1=1 --
' AND 1=2 --
```

If the `1=1` request returns the normal page but `1=2` returns a different page (or missing content), the input likely reaches a conditional in SQL.

Character-by-character extraction (example pattern):

```sql
' AND SUBSTRING((SELECT user()),1,1) = 'a' --
```

Iterate characters and positions to extract values using many requests.

What to measure:

* Presence/absence of specific DOM elements (e.g., login form, item details).
* Response length differences (byte count).
* Status codes or redirect behaviors.

Mitigation notes:

* Normalize error and content outputs so boolean differences aren’t observable.
* Use parameterized queries to remove conditional injection points.

---

## 2. Time-based detection

**Idea:** Use DB functions that delay execution (e.g., `SLEEP()`, `pg_sleep()`) to infer true/false by measuring response time.

Example payloads:

```sql
' OR (SELECT CASE WHEN (SUBSTRING(version(),1,1)='5') THEN SLEEP(5) ELSE 0 END) --
```

Or a simpler timing test:

```sql
' OR IF(1=1,SLEEP(5),0) --   -- MySQL
' OR CASE WHEN 1=1 THEN pg_sleep(5) ELSE pg_sleep(0) END -- Postgres
```

If the response is consistently delayed by \~5 seconds for the true condition, the endpoint is likely vulnerable to time-based blind SQLi.

What to watch for:

* Network jitter and variability — repeat tests and measure averages.
* Application-level timeouts or request queuing that may affect measurements.

Mitigation notes:

* Disable or restrict expensive DB functions for app accounts.
* Implement request timeouts and rate limiting.

---

## 3. Response fingerprinting & subtle indicators

Sometimes differences are very subtle; consider:

* Tiny differences in HTML (extra whitespace or different attribute values).
* JSON responses with slightly different fields or boolean flags.
* Cookies or headers set only on successful queries.

Use automated diffing tools or scripts to compare responses programmatically.

---

## 4. Automation & tooling (authorized use only)

* Burp Suite Intruder / Repeater for manual testing and diffs.
* sqlmap for automating detection and exploitation (use carefully and only where allowed).
* Custom scripts (Python + `requests`) to measure response times and lengths.

Example detection script idea (pseudo): send baseline request, then send crafted `true` and `false` payloads, compare status/length/time.

---

## 5. Defensive guidance

* Parameterize queries and avoid concatenation.
* Consistent responses: return the same page/structure for different internal code paths when feasible.
* Implement WAF rules to detect repeated probing patterns, unusual parameter lengths, or high request rates.
* Monitor logs for many similar queries probing for boolean/time patterns.

---

## 6. Practical tips for testers

* Always capture a baseline response before testing.
* Avoid noisy payloads in production; use low-frequency probing.
* Prefer lab environments for extraction exercises — blind extraction can generate thousands of requests.