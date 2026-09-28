# OWASP Juice Shop — Security Assessment

Web application penetration test of **[OWASP Juice Shop](https://owasp.org/www-project-juice-shop/)**, the intentionally vulnerable application maintained by OWASP for security training and research.

This repository documents a full black-box + gray-box assessment: **107 vulnerabilities** identified, exploited and mapped to the OWASP Top 10, with clean, reproducible write-ups.

> **Author:** Daladhier Vásquez ([@Daladh](https://github.com/Daladh))
> **Target:** OWASP Juice Shop (public training lab)
> **Type:** Educational / portfolio security research

---

## ⚠️ Scope & ethics

All testing was performed against a **local instance of OWASP Juice Shop**, a deliberately vulnerable application published by OWASP specifically for learning. No third-party, production or client systems were touched. Every technique here is intended for **authorized, educational use only**.

---

## 📊 Assessment at a glance

| Severity | Count |
|----------|-------|
| 🔴 Critical | 22 |
| 🟠 High | 44 |
| 🟡 Medium | 26 |
| 🟢 Low | 13 |
| ⚪ Informational | 2 |
| **Total** | **107** |

Coverage spans the full OWASP Top 10: broken access control, cryptographic failures, injection (SQL, NoSQL, XSS, SSTI, XXE), insecure design, security misconfiguration, vulnerable components, authentication failures, and SSRF — plus Web3/blockchain and business-logic flaws unique to the target.

Full list: **[Vulnerability Matrix →](vulnerability-matrix.md)**

---

## 🔬 Featured write-ups

Detailed, reproducible walkthroughs of representative findings:

| # | Write-up | Category | Severity |
|---|----------|----------|----------|
| 01 | [SQL Injection — Admin Login Bypass](writeups/01-sqli-login-bypass.md) | Injection | 🔴 Critical |
| 02 | [Forged JWT — Authentication Bypass](writeups/02-jwt-forged-token.md) | Auth failures | 🔴 Critical |
| 03 | [XXE — External Entity Data Access](writeups/03-xxe-data-access.md) | Injection | 🔴 Critical |
| 04 | [SSRF via Profile Image URL](writeups/04-ssrf-profile-image.md) | SSRF | 🟠 High |
| 05 | [NoSQL Injection — Denial of Service](writeups/05-nosql-injection-dos.md) | Injection | 🟠 High |
| 06 | [Broken Access Control — Other Users' Baskets](writeups/06-broken-access-control-basket.md) | Access control | 🟡 Medium |

---

## 🧭 Methodology

The engagement followed a standard web application penetration testing workflow, aligned with the **OWASP Web Security Testing Guide (WSTG)** and **PTES**:

1. **Reconnaissance & mapping** — enumerate endpoints, JS bundles, API routes, hidden paths.
2. **Authentication & session analysis** — JWT handling, password recovery, session management.
3. **Access control testing** — horizontal/vertical privilege escalation, IDOR.
4. **Injection testing** — SQL, NoSQL, XSS, SSTI, XXE across every input.
5. **Business logic** — payment flows, coupons, basket manipulation, Web3 features.
6. **Sensitive data exposure** — OSINT, exposed files, hardcoded secrets, metadata.
7. **Reporting** — CVSS scoring, business impact, remediation guidance.

## 🧰 Tooling

`Burp Suite` · `OWASP ZAP` · `sqlmap` · `Nmap` · `ffuf` · `jwt_tool` · Browser DevTools · custom scripts

---

## 📁 Repository structure

```
.
├── README.md                 # This file
├── vulnerability-matrix.md   # Full list of all 107 findings
└── writeups/                 # Detailed technical write-ups
```

---

## 📜 License

Documentation released under the [MIT License](LICENSE). OWASP Juice Shop is © OWASP Foundation, used here strictly for educational purposes.
