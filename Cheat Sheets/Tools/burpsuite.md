# Burp Suite — Quick Reference for SQL Injection Testing

> [!WARNING]
> **Ethics reminder:** Use Burp Suite **only** on authorized systems or lab environments. Unauthorized testing is illegal and unethical.

---

## What is Burp Suite?

Burp Suite is a web security testing platform that helps identify vulnerabilities, including SQL Injection. It allows interception, modification, and analysis of HTTP/S traffic between the client and server.

---

## Key Features for SQLi Testing

1. **Proxy**: Intercept HTTP requests/responses.
2. **Repeater**: Manually manipulate requests to test for SQLi.
3. **Scanner (Professional)**: Automated detection of SQLi and other vulnerabilities.
4. **Intruder**: Automated payload injection with custom or built-in payload lists.
5. **Logger / HTTP history**: Track all traffic for analysis.

---

## Manual SQLi Testing Workflow

1. **Intercept a request** via Burp Proxy.
2. **Send to Repeater**: Identify input parameters.
3. **Inject simple payloads**:

   * `'` or `"` for syntax errors
   * `OR 1=1` / `AND 1=2` for boolean tests
4. **Observe responses**: Look for error messages, differences in content, or timing changes.
5. **Iterate** with additional payloads and techniques (Union-based, Blind, Time-based).

---

## Burp Intruder SQLi Testing

* Set payload positions around user-supplied parameters.
* Use built-in payload lists or custom lists with SQLi probes.
* For Boolean-based blind testing, monitor response length or content.
* For Time-based blind testing, include payloads with delays and watch response times.
* Avoid high-frequency attacks on production systems; limit request rate.

---

## Safe Practices

> [!IMPORTANT]
> * Use only in lab or authorized scope.
> * Document each request/response for reporting.
> * Start with non-destructive payloads.
> * Confirm findings with multiple payload types.

---

## Tips & Tricks

> [!TIP]
> * Use `Repeater` to manually test hypotheses before automating in `Intruder`.
> * Save all traffic for analysis; you can filter by parameter names or HTTP status codes.
> * Combine with `sqlmap` for automation after confirming injection manually.
> * Leverage Burp Extensions for enhanced SQLi testing, e.g., `SQLiPy` or `SQLiPy-ng`.

---

## Resources

* Burp Suite Academy (PortSwigger)
* OWASP Web Security Testing Guide (WSTG)
* Practice labs: DVWA, OWASP Juice Shop, PortSwigger Academy