# sqlmap — Quick Reference & Safe Usage Guide

> [!WARNING]
> **Ethics & safety:** `sqlmap` is a powerful automated SQL Injection tool. Use it **only** in authorized environments (your lab, explicit penetration-test scope, or bug bounty programs that permit such testing). Running `sqlmap` against systems without permission is illegal and unethical.

---

## What is sqlmap?

`sqlmap` is an open-source penetration testing tool that automates detection and exploitation of SQL Injection vulnerabilities. It supports many DBMS types (MySQL, PostgreSQL, MSSQL, Oracle, etc.) and offers techniques for fingerprinting, data extraction, and some out-of-band and OS-level interactions.

This page is a concise cheat-sheet for common, non-destructive `sqlmap` usage patterns and important flags. Always review and tailor commands to the scope of your engagement.

---

## Common safe flags and meanings

* `-u <url>` — Target URL (required for basic usage).
* `--data="<POSTDATA>"` — Use for POST requests.
* `-p <param>` — Specify parameter(s) to test (e.g., `-p id`).
* `--cookie="SESSION=..."` — Provide session cookies for authenticated testing.
* `--batch` — Non-interactive mode (use cautiously; it accepts defaults automatically).
* `--level=<1-5>` — Test depth; higher levels test more payloads and are slower/noisier. Start at `--level=1`.
* `--risk=<0-3>` — Risk level of payloads to try (higher risk may use more disruptive techniques). Start with `--risk=1`.
* `--technique=<T>` — Force specific techniques: `B` (boolean), `E` (error), `U` (union), `S` (stacked), `T` (time), `O` (OOB). Example: `--technique=BE`.
* `--dbms=<dbms>` — Limit testing to a specific DBMS (e.g., `MySQL`, `PostgreSQL`) to speed up testing.
* `--timeout=<secs>` — Set request timeout to handle slow targets.
* `--threads=<n>` — Number of concurrent threads; be conservative in production-like environments.

---

## Safe discovery workflow (recommended)

1. **Get written permission** and confirm rules of engagement (time windows, scope, excluded hosts).
2. **Fingerprint manually first**: send a few simple probes (`'`, `AND 1=1/1=2`) to observe behavior.
3. **Run sqlmap with cautious settings**:

   * Start with `--level=1 --risk=1 --batch -p <param>`.
   * Limit `--threads` to a small number (e.g., `--threads=1` or `2`).
   * Use `--dbms` if you already know the backend to reduce noise.
4. **Upgrade depth cautiously**: increase `--level` and `--risk` only if authorized and necessary.
5. **Prefer non-destructive options**: avoid flags that attempt to write to the DB or get a shell (e.g., `--os-shell`, `--os-pwn`, `--os-smbrelay`) unless explicitly authorized.

---

## Non-destructive examples

* Basic test on `id` parameter (GET):

```bash
sqlmap -u "https://example.com/item.php?id=1" -p id --batch --level=1 --risk=1
```

* Authenticated POST test:

```bash
sqlmap -u "https://example.com/login.php" --data="username=admin&password=foo" -p username --cookie="PHPSESSID=abc" --batch
```

* Use boolean and time techniques only after confirmation:

```bash
sqlmap -u "https://example.com/item.php?id=1" -p id --technique=BT --batch --level=2 --risk=1
```

* Limit to MySQL to speed up testing:

```bash
sqlmap -u "https://example.com/item.php?id=1" -p id --dbms=MySQL --batch
```

---

## Flags to avoid unless explicitly authorized

* `--os-shell`, `--os-pwn`, `--os-smbrelay` — attempt to get OS-level shells; highly invasive.
* `--dump` on sensitive tables without permission — only dump data within scope.
* `--threads` very high — can DoS the target.
* `--flush-session` — clears saved sessions; use carefully when running multiple tests.

---

## Interpreting results & follow-ups

* If `sqlmap` reports injection, verify findings manually using low-noise, non-destructive probes.
* Capture request/response history (e.g., with Burp) and include sanitized examples in your report.
* Recommend fixes: parameterized queries, input validation, least privilege, and hiding detailed errors.

---

## Defensive guidance for defenders

> [!TIP]
> * Use parameterized queries and ORM/query builders correctly.
> * Rate-limit and add WAF protections as an additional layer.
> * Monitor logs for many similar requests, long query execution patterns, or `sqlmap`-like fingerprinting behavior.
> * Harden DBMS accounts and restrict dangerous features and functions.

---

## Resources & testing labs

* PortSwigger Academy, OWASP Juice Shop, DVWA — practice safely with `sqlmap` on lab targets.
* Always document your testing methodology and obtain permission before running `sqlmap` against real systems.