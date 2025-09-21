# SQL Injection Cheat Sheet 📘

![GitHub last commit](https://img.shields.io/github/last-commit/JaberAlJ/sqli-cheat-sheet)
![GitHub stars](https://img.shields.io/github/stars/JaberAlJ/sqli-cheat-sheet?style=social)

A concise, practical reference for learning, detecting, and safely testing **SQL Injection (SQLi)** vulnerabilities.

> ✍️ Written by: [*JaberAlJ*](https://github.com/JaberAlJ)

---

> [!WARNING]
> #### **Ethics & safe use**
> This repository is intended **only** for educational purposes, secure development, and authorized security testing. Do **not** use the materials here to attack systems you do not own or do not have explicit permission to test. Unauthorized access is illegal and unethical.
>
>If you are new to security testing, get explicit written permission (a scope and rules of engagement) from the owner before testing. Always follow local laws and organizational policies.

---

## What this repository contains

The repo is organized to mirror how many security learning resources (e.g. PortSwigger) present SQLi concepts: from fundamentals to detection, exploitation, and advanced techniques, with platform-specific examples and tool guidance.

Top-level folders and purpose:

* [Basics](Cheat%20Sheets/Basics/README.md) — Introduces SQLi, common SQL syntax and statements you should know.
* [Detection](Cheat%20Sheets/Detection/README.md) — Safe detection methods: error-message analysis, response analysis, and techniques for recognizing blind SQLi.
* [Exploitation](Cheat%20Sheets/Exploitation/README.md) — Core exploitation techniques (UNION, error-based, boolean/time-based blind, stacked queries).
* [Advanced](Cheat%20Sheets/Advanced/README.md) — Out-of-band techniques, second-order SQLi, and filter/WAF bypass strategies.
* [Examples](Cheat%20Sheets/Examples/README.md) — Concrete examples and payloads for MySQL, MSSQL, Oracle, PostgreSQL, etc.
* [Tools](Cheat%20Sheets/Tools/README.md) — How to use common tools (sqlmap, Burp Suite) responsibly, plus curated payload lists.

Each folder contains a `README.md` explaining its scope and linking to the files inside.

---

## Who this is for

* Developers who want to understand how SQLi works so they can fix it.
* Security students and beginners learning web application security concepts.
* Penetration testers who need a compact reference of techniques and payloads.

---

## How to use this cheat sheet

1. **Start in [Basics](Cheat%20Sheets/Basics/README.md)** to make sure you understand SQL syntax and the different types of SQLi.
2. **Read [Detection](Cheat%20Sheets/Detection/README.md)** to learn how to identify vulnerable inputs safely without causing harm.
3. **Move to [Exploitation](Cheat%20Sheets/Exploitation/README.md)** only in controlled, authorized environments to practice how payloads behave.
4. **Refer to [Advanced](Cheat%20Sheets/Advanced/README.md)** to learn advanced topics.
5. **Consult [Examples](Cheat%20Sheets/Examples/README.md)** for DBMS-specific quirks and realistic payloads you can practice with locally or on intentionally vulnerable labs.
6. **Use [Tools](Cheat%20Sheets/Tools/README.md)** for automation and proof-of-concept testing — always aligned with your authorization.

---

## 🚀 Getting Started
Browse the [📂 Cheat Sheets](./Cheat%20Sheets) folder and open any `.md` file to view commands.  
Each section is self-contained and focuses on a specific SQLi topic.

---

## ⭐ Support
If you find this project helpful, please give it a star ⭐ on GitHub — it helps others discover it!

## 👋 Contribution

Contributions are always welcome! If you find any commands that are missing, incorrect, or could be improved, please feel free to:

1. **Open an Issue**: Report a bug, suggest an enhancement, or ask a question.
2. **Submit a Pull Request**: Make changes directly and submit them for review.

---

## Attribution & resources

> [!NOTE]
> This repository is informed by widely used public resources, tutorials, and practical write-ups in the field of SQL Injection and web application security. The following references were used or recommended for further learning:

### Learning Platforms
- [PortSwigger Web Security Academy](https://portswigger.net/web-security) — Interactive labs and comprehensive tutorials on SQLi and other web vulnerabilities.
- [OWASP Web Security Testing Guide (WSTG)](https://owasp.org/www-project-web-security-testing-guide/) — Authoritative guidance for testing web application security.
- [OWASP Juice Shop](https://owasp.org/www-project-juice-shop/) — Vulnerable web application for safe hands-on practice.

### Practical Tutorials & Write-ups
- [SQL Injection Explained](https://www.acunetix.com/websitesecurity/sql-injection/) — Clear explanations with examples of SQLi types and attacks.
- [SQLi Labs & Exercises](https://github.com/Audi-1/sqli-labs) — Lab exercises for MySQL, PostgreSQL, and other databases.
- [PortSwigger SQLi Cheat Sheet](https://portswigger.net/web-security/sql-injection/cheat-sheet) — Reference for payloads and techniques.

### Reference Material
- Database documentation for MySQL, MSSQL, PostgreSQL, Oracle — for understanding system functions and version-specific quirks.
- Security write-ups and research articles from Bug Bounty programs (publicly disclosed).

> [!WARNING]
> All materials are intended for educational purposes. Always use labs or authorized environments for testing and follow ethical guidelines.

---

## 📂 Repository Structure

The repository is organized into logical sections to make it easy to find the SQLi statements you need.  
Each topic has its own dedicated Markdown file (`.md`) inside the `Cheat Sheets/` directory.

```text
├── 📁 Cheat Sheets/
│   ├── 📁 Advanced/
│   │   ├── 📖 README.md
│   │   ├── 📝 bypass-filters.md
│   │   ├── 📝 out-of-band.md
│   │   └── 📝 second-order.md
│   ├── 📁 Basics/
│   │   ├── 📖 README.md
│   │   ├── 📝 sql-injection-types.md
│   │   ├── 📝 sql-statements.md
│   │   └── 📝 sql-syntax.md
│   ├── 📁 Detection/
│   │   ├── 📖 README.md
│   │   ├── 📝 blind-sqli.md
│   │   ├── 📝 error-messages.md
│   │   └── 📝 response-analysis.md
│   ├── 📁 Examples/
│   │   ├── 📖 README.md
│   │   ├── 📝 mssql-examples.md
│   │   ├── 📝 mysql-examples.md
│   │   ├── 📝 oracle-examples.md
│   │   └── 📝 postgresql-examples.md
│   ├── 📁 Exploitation/
│   │   ├── 📖 README.md
│   │   ├── 📝 boolean-based.md
│   │   ├── 📝 error-based.md
│   │   ├── 📝 stacked-queries.md
│   │   ├── 📝 time-based.md
│   │   └── 📝 union-based.md
│   └── 📁 Tools/
│       ├── 📖 README.md
│       ├── 📝 burpsuite.md
│       ├── 📝 payloads.md
│       └── 📝 sqlmap.md
└── 📖 README.md
```