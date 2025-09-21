# Examples — DBMS-specific SQLi Examples & Notes

This folder gathers DBMS-specific, non-destructive SQL Injection examples and guidance to help you practice and understand how attacks vary across database engines. 

> [!IMPORTANT]
> All examples are educational — **only use them in authorized test environments or labs**.

## Files in this folder

* [mysql-examples.md](mysql-examples.md) — MySQL-specific probes, UNION/error/boolean/time patterns, and MySQL functions.
* [mssql-examples.md](mssql-examples.md) — Microsoft SQL Server examples, `WAITFOR` timing, `@@version`, and notes on extended procedures.
* [postgresql-examples.md](postgresql-examples.md) — PostgreSQL examples using `pg_sleep`, system catalogs, and Postgres-specific functions.
* [oracle-examples.md](oracle-examples.md) — Oracle examples, PL/SQL notes, `DBMS_LOCK.SLEEP`, `UTL_HTTP`, and privilege caveats.

## How to use these files

1. **Start with detection probes** in a controlled lab to confirm injection influence.
2. **Use non-destructive enumeration** (e.g., `version()`, `user()`, `database()`) to fingerprint the DBMS.
3. **Move to blind techniques** (boolean/time) only when direct output is unavailable.
4. **Always prefer parameterized queries and defensive testing** when practicing.

## Practice & safety

> [!TIP]
> * Use intentionally vulnerable labs (PortSwigger Academy, OWASP Juice Shop, DVWA) or local Docker/VM setups.
> * Never run destructive commands on production systems; prefer safe probes and document all tests.