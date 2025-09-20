# Detecting SQL Injection via Error Messages

Error messages can provide attackers with valuable clues about whether an application is vulnerable to SQL Injection. As defenders, we must understand these signals so we can eliminate them in production and recognize them during authorized testing.

> [!WARNING]
> These techniques should only be used in safe lab environments or with explicit permission during authorized penetration tests. Never attempt them on systems you don’t own.

---

## 1. Why error messages matter

* They often expose **SQL syntax**, **table/column names**, or **DBMS type/version**.
* Different DBMS produce distinctive error messages, allowing attackers to fingerprint the backend.
* Verbose error handling in development environments is a common source of leaks.

---

## 2. Common database error patterns

### MySQL

* `You have an error in your SQL syntax; check the manual...`
* `Unknown column 'abc' in 'where clause'`

### Microsoft SQL Server (MSSQL)

* `Unclosed quotation mark after the character string...`
* `Incorrect syntax near '...'`
* `Conversion failed when converting the varchar value ... to data type int`

### Oracle

* `ORA-00933: SQL command not properly ended`
* `ORA-01756: quoted string not properly terminated`
* `ORA-00904: invalid identifier`

### PostgreSQL

* `ERROR: syntax error at or near "..."`
* `ERROR: column "abc" does not exist`
* `ERROR: operator does not exist: integer = text`

---

## 3. Simple payloads to trigger error-based detection

These payloads are commonly used in detection (not exploitation):

* `'` (single quote) → often causes syntax error if not properly escaped.
* `"` (double quote) → depending on DBMS quoting rules.
* `;` (semicolon) → may terminate a statement and cause an error if multiple statements aren’t allowed.
* `OR 1=1--` → may change query logic; errors differ depending on context.

Example:

```http
GET /products?id=1'
```

Response may show: `You have an error in your SQL syntax...`

---

## 4. Defensive strategies

* **Hide error messages from users**: Replace with generic messages like “An error occurred.”
* **Log errors internally**: Store detailed DB errors in server logs for developers/admins.
* **Use parameterized queries**: Prevent unescaped input from altering query structure.
* **Input validation**: Reject unexpected characters early (quotes, semicolons, comment markers).
* **Environment hardening**: Run production with verbose errors disabled.

---

## 5. Key takeaway

Verbose database error messages are a major detection vector for SQL Injection. As defenders, we must:

* Catch and sanitize them.
* Prevent SQL Injection through parameterization and least-privilege access.
* Recognize them in testing to identify vulnerable endpoints early.