# Tools — SQL Injection Testing Utilities

This folder contains guides for practical tools used in SQL Injection testing, along with safe usage examples and best practices. 

> [!IMPORTANT]
> All tools should be used **only in authorized testing environments or labs**.

## Files in this folder

* `burpsuite.md` — Guidance for using Burp Suite to manually test SQL Injection vulnerabilities.
* `payloads.md` — Common SQLi payloads for safe testing, including detection, UNION/error/boolean/time-based payloads and safe stacked-query examples.
* `sqlmap.md` — Quick reference for `sqlmap`, safe flags, non-destructive examples, and defensive guidance.

## How to use these files

1. **Start with `sqlmap.md`** if you want to automate testing.
2. **Use `burpsuite.md`** for manual analysis, Repeater/Intruder usage, and observing request/response behaviors.
3. **Reference `payloads.md`** for educational payloads and safe test cases.

## Safety tips

> [!TIP]
> * Always operate within a lab or authorized scope.
> * Prefer non-destructive queries first.
> * Document all experiments and review tool outputs before acting.
> * Combine multiple tools for verification rather than relying solely on automated detection.