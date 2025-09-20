# Detection — Overview & How to Use

This folder contains techniques and guidance for **safely detecting** SQL Injection vulnerabilities without causing unnecessary harm. The focus is on identification methods that help you confirm whether input influences SQL queries — and on how defenders can harden systems against these detection vectors.

## Recommended reading order

1. [error-messages.md](error-messages.md)
2. [response-analysis.md](response-analysis.md)
3. [blind-sqli.md](blind-sqli.md)

## Files in this folder

* [blind-sqli.md](blind-sqli.md) — Blind SQLi techniques with examples and defensive countermeasures. Detailed guidance for blind SQLi detection and extraction (boolean-based and time-based techniques).
* [error-messages.md](error-messages.md) — DBMS-specific error patterns and safe detection payloads. Learn what database error messages look like and how attackers use them to fingerprint and discover injection points.
* [response-analysis.md](response-analysis.md) — How to measure and compare responses; automation tips and mitigation. Techniques for detecting SQLi by comparing responses (status codes, content, headers, timing).

## How to use these pages

* Use [error-messages.md](error-messages.md) when the application returns visible errors or stack traces.
* Use [response-analysis.md](response-analysis.md) when errors are suppressed — it explains how to spot differences in content, headers, and timing.
* Use [blind-sqli.md](blind-sqli.md) for step-by-step blind exploitation techniques **only in authorized environments**.

## Defensive checklist for detection vectors

* Hide verbose DB errors and stack traces in production.
* Standardize responses to make boolean differences less obvious.
* Restrict expensive DB functions and monitor for abnormal timing patterns.
* Implement Web Application Firewall (WAF) rules to catch common probing payloads.