# Bypassing Filters & WAFs — Techniques, Caveats, and Defensive Notes

> [!WARNING]
> **Ethics reminder:** The techniques below describe methods attackers use to evade input filters and WAF rules. This information is provided for **defensive** purposes — to help developers and defenders understand attack patterns and harden systems. Use these techniques only in authorized testing environments and never to attack systems without explicit permission.

---

## Why filters and WAFs can be bypassed

* WAFs and filters apply pattern matching, heuristics, or signatures — they can be evaded by altering payload appearance while keeping its semantic effect.
* Application-layer validation may only check a canonical representation; encoded or obfuscated input can slip through.
* DBMS-specific quirks, alternate encodings, and different parsing stages (app → DB driver → DB) create opportunities for mismatch-based bypasses.

---

## Common evasion techniques (illustrative)

Below are common techniques attackers use. Defenders should know them to design robust protections and tests.

### 1. Alternate encodings

* **URL-encoding / double URL-encoding:** Encode payload characters (`%27` for `'`) — sometimes filters decode only once.
* **Hex/Unicode encodings:** Use `0x...` hex literals (MySQL), `CHAR()` functions, or Unicode escapes to represent characters.

  * Example (MySQL): `CHAR(0x27)` or `0x27` equivalents to `'`.

### 2. Inline comments & whitespace variation

* Insert comments to split keywords: `UN/**/ION` or `UNI--
  ON` to bypass naive patterns.
* Use alternative whitespace (tab, newline) or comment tricks supported by DBMS.

### 3. Case variation and spacing

* SQL is case-insensitive for keywords; using different cases (`UniOn`, `SeLeCt`) can bypass simple signature checks.
* Insert extra spaces: `UN ION SELECT` in some contexts, or use unusual Unicode whitespace.

### 4. Function-based obfuscation

* Build strings with `CHAR()`/`CHR()` functions and concatenate to form keywords or data: `CONCAT(CHAR(117),CHAR(110),CHAR(105),CHAR(111),CHAR(110))` → `union`.
* Use database functions to compute values instead of literal payload parts.

### 5. Query fragmentation & partial injection

* Break payload across multiple parameters or fields that get concatenated on the server side (second-order techniques).
* Use stored procedures or multi-stage execution paths to hide the malicious part until later.

### 6. Using alternative SQL constructs

* Replace `UNION` with `UNION ALL` or use `SELECT` variations that achieve the same result with different fingerprints.
* Use DBMS-specific functions or system tables to retrieve data without using signature keywords.

### 7. Leveraging DBMS quirks and functions

* Use DBMS-specific functions that perform similar actions but are less commonly blocked (e.g., `pg_sleep`, `WAITFOR`, `DBMS_PIPE`, `UTL_HTTP`).
* Use implicit conversions, concatenations, or odd operator precedence to alter payload appearance.

### 8. Encoded payloads in headers or other vectors

* Place payloads in HTTP headers (e.g., `User-Agent`, `Referer`) if application logs or header values are used in SQL queries.
* Use multipart/form-data boundaries or JSON structures to hide patterns from parsers that only inspect URL/query parameters.

### 9. Time-based and blind techniques (indirect bypass)

* If WAF blocks obvious payloads, use blind techniques (boolean/time-based) that do not include blocked keywords and are harder to detect.
* Use function-based conditions (`IF`, `CASE`) combined with delays to infer data slowly.

### 10. Exploiting inconsistent normalization

* Differences in how the app normalizes input vs. how the DB parses it (e.g., Unicode normalization, trimming, or character folding) can be exploited by encoding payloads that the app considers safe but the DB executes differently.

---

## Examples (non-executable, defensive examples)

* Bypass `--` comment filter: `';/*--*/ SELECT ...` or `'+CHAR(13)+CHAR(10)+'--` inserted to change how the payload is tokenized.
* Build `UNION` using `CHAR()` in MySQL: `UNION SELECT NULL, CONCAT(CHAR(118),CHAR(101),CHAR(114),CHAR(115),CHAR(105),CHAR(111),CHAR(110)) --` to reveal `version`.
* Use hex-encoded string: `0x61646d696e` (represents `admin` in hex) in contexts where hex literals are accepted.

---

## Defensive measures — how to reduce bypass risk

Understanding attack patterns suggests defensive measures. Combine multiple controls for depth.

### 1. Primary defense: Parameterized queries

* Prepared statements and parameter binding remove the problem entirely for injection in data contexts (the strongest defense).

### 2. Canonicalization & normalization

* Normalize input consistently (Unicode normalization, trimming) before validation, and validate canonical forms.
* Apply output encoding appropriate to the context (HTML, JavaScript, SQL identifiers — if unavoidable).

### 3. Positive validation (whitelisting)

* Prefer whitelist validation (allowed characters, formats) over blacklists, especially for structured inputs (IDs, emails, column names).

### 4. Limit dangerous functionality

* Disable or restrict DB features that can be abused (e.g., `xp_cmdshell`, `UTL_HTTP`, `DBMS_PIPE`, `pg_sleep` if not required).
* Run DB services in networks without direct egress to the internet when possible.

### 5. WAF & IDS tuning (defensive, not primary)

* Use WAFs to block common automated attacks and noisy payloads, but don’t rely on them as the only protection.
* Regularly update WAF rules and tune to reduce false positives/negatives.

### 6. Logging, monitoring & anomaly detection

* Log queries and inputs (safely) and monitor for patterns: repeated probes, many similar requests, or odd encodings.
* Alert on unusual combinations (e.g., many requests containing `CHAR()` or long hex literals).

### 7. Code review & static analysis

* Review code for dynamic SQL construction (string concatenation, `EXEC`, `eval`) and apply fixes.
* Use static analysis tools that detect risky patterns and taint flows from input to SQL sinks.

### 8. Least privilege & segmentation

* Ensure application DB accounts have only required permissions.
* Segment databases and block unnecessary outbound network access to prevent OOB exfiltration.

---

## Testing recommendations for defenders

> [!TIP]
> * Simulate evasions in a lab environment: test URL-encoded, double-encoded, comment-inserted, and function-built payloads against your WAF and app logic.
> * Use mutation/fuzzing approaches (Burp Suite, custom scripts) to find normalization or parsing mismatches.
> * Combine code review (find concatenation sinks) with live testing targeted at those sinks.

---

## Final notes

> [!NOTE]
> Filter and WAF evasion is a cat-and-mouse game. The strongest, most reliable defenses are secure coding practices (parameterised queries, input validation, least privilege) combined with robust monitoring and careful WAF tuning. Understand the techniques not to exploit systems, but to design resilient applications and detection rules.