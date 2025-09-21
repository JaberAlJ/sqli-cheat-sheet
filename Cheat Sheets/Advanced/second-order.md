# Second‑Order SQL Injection

> [!WARNING]
> **Ethics reminder:** Second‑order SQL Injection is subtle — payloads are stored by the application and later executed in a different context. Use these techniques only in authorized testing environments and with permission. Do not exploit systems you do not own.

---

## What is Second‑Order SQL Injection?

Second‑order SQLi happens when malicious input is **stored** by the application (e.g., in a database) without causing immediate harm, but that stored data is later used in a different query or context **without proper sanitization**, leading to injection.

This differs from classic (first‑order) SQLi, where the malicious input is immediately reflected into a query and causes an immediate effect. Second‑order often bypasses naive input filters because the initial storage appears harmless.

---

## Typical scenarios

* **Stored profile fields:** An attacker enters a value in a profile form (e.g., `display_name`) that gets saved. Later, an admin page or reporting function builds a SQL query using that stored value without parameterization.
* **Escaped-at-insert, concatenated-at-use:** Application escapes input for initial storage but later concatenates the stored value into a different query where escaping is ineffective.
* **Multi-step workflows:** Data flows through multiple application layers (input → storage → processing) and sanitization is applied inconsistently at different stages.

Example flow:

1. User registers with `bio` field containing: `O'Reilly` or a payload like `'; UPDATE users SET role='admin' WHERE id=1; --`.
2. The app stores this value in the DB (no immediate error).
3. Later, an admin interface runs: `SELECT * FROM reports WHERE note = '" + bio + "'` — the stored payload now alters the query and executes.

---

## Example payloads & patterns (conceptual)

* Stored string that contains a terminator and subsequent statement: `'; SELECT ... --` (works only if the later context concatenates into SQL).
* Stored value causing type mismatches or logic changes when used in different contexts (e.g., numeric context vs string context).

**Important:** Concrete payloads depend on how the stored data is later used. Effective testing requires mapping application data flows.

---

## How to test for second‑order SQLi (authorized)

1. **Map data flows:** Identify where user input is stored and where that stored data is later used in queries or templates.
2. **Store benign markers first:** Insert unique, non-destructive markers (e.g., `INJECT-XYZ-123`) instead of dangerous payloads to trace where the stored content is used.
3. **Observe sinks:** Find pages, reports, or backend jobs that read the stored value — these are potential sinks.
4. **Craft targeted payloads at the sink context:** If the sink builds SQL queries using the stored value, craft payloads matching that context (quoted vs numeric, part of identifiers, etc.).
5. **Use safe, non-destructive tests** before escalating to any destructive or noisy payloads.

**Example test:**

* Store `INJECT_MARKER` in a profile field.
* Search admin pages or logs for `INJECT_MARKER` to see where the value is used.
* If found in an SQL-building context, try context-appropriate probe payloads under authorization.

---

## Detection tips

* **Instrumented logging:** When authorized, add logging around sinks to see how stored values are incorporated into queries.
* **Search codebase:** Look for code patterns that concatenate DB fields into SQL (e.g., `"..." + value + "..."`), dynamic SQL builders, or `EXEC`/`EXECUTE` usages that consume stored data.
* **Review templates and reporting code:** Many second‑order vulnerabilities surface in reporting, admin, or batch-processing code rather than public-facing input handlers.

---

## Defensive measures

1. **Parameterize all queries at every sink** — never trust stored data; treat it as untrusted when used in SQL.
2. **Consistent sanitization strategy:** Apply canonicalization and validation both at input and before usage, but rely on parameterization as the primary defense.
3. **Avoid delayed concatenation:** Don’t build SQL by concatenating data at runtime — use prepared statements, stored procedures with bind parameters, or safe query builders.
4. **Code reviews & dataflow analysis:** Regularly audit code paths where stored user-controlled values are used in SQL or in contexts that interpret content (e.g., HTML, LDAP, shell commands).
5. **Least privilege:** Even if exploitation occurs, limited DB privileges can reduce impact.

---

## Real‑world examples & red flags

* An e‑commerce site saves a `search_query` or `sort_by` preference and later uses it directly inside `ORDER BY` or `LIMIT` clauses — if not checked, this can allow injection into those clauses.
* Admin export features that dynamically assemble SQL using stored report filters or saved searches.

**Red flags in code reviews:**

* Dynamic SQL built from DB-stored fields.
* Use of `EXEC`, `sp_executesql`, `eval`, or similar runtime execution functions with concatenated input.

---

## Lab practice recommendations

* Build a small multi-page app where input is stored and later used in an admin report. Practice injecting markers and tracing flows to simulate second‑order discovery.
* Use static analysis or code search tools (grep, ripgrep) to find potential sinks in sample codebases.

---

## Final notes

> [!NOTE]
> Second‑order SQLi is subtle because the injection vector and the execution sink are separated in time and code paths. Defending against it requires a defensive mindset: treat stored data as untrusted wherever it is later used. Automated scanning can miss these issues — combine code review, dataflow analysis, and targeted testing.