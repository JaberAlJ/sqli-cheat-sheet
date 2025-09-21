# Advanced — Overview & How to Use

This folder contains advanced SQL Injection topics that require deeper understanding of application and DB internals. The content focuses on less-common but higher-impact techniques, how they work, and defensive controls.

## Files in this folder

* [bypass-filters.md](bypass-filters.md) — Evasion techniques, examples, and defensive recommendations. Techniques attackers use to evade filters and WAFs; important for defenders to test against.
* [out-of-band.md](out-of-band.md) — DNS/HTTP exfiltration, DBMS functions, and network defenses. OOB exfiltration using DNS/HTTP when direct responses are unavailable.
* [second-order.md](second-order.md) — Second-order injection scenarios, detection, and defenses. Learn about stored (second-order) SQLi where malicious input is executed later in a different context.

## Recommended reading order

1. [second-order.md](second-order.md)
2. [bypass-filters.md](bypass-filters.md)
3. [out-of-band.md](out-of-band.md)

## How to use these pages

* Read `second-order.md` if your application stores user input and later uses it in SQL queries or admin reports.
* Use `bypass-filters.md` to understand how attackers may evade signature-based protections and to design robust tests.
* Read `out-of-band.md` when investigating ways attackers might exfiltrate data via DNS/HTTP; focus on network-level controls to mitigate.

## Defensive checklist (advanced)

> [!TIP]
> * Treat stored values as untrusted at every sink and parameterize queries at use-time.
> * Restrict or disable dangerous DB functions and packages for application accounts.
> * Implement strict egress controls for DB/application servers and monitor outbound DNS/HTTP activity.
> * Combine code review, dataflow analysis, and mutation testing to find subtle injection paths.