# 01 — SQL Injection: Admin Login Bypass

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1** | 9.8 (`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`) |
| **Category** | Injection (OWASP A03:2021) |
| **Endpoint** | `POST /rest/user/login` |

## Summary

The login endpoint builds its SQL query by concatenating user-supplied input directly into the statement, without parameterization. An attacker can inject SQL through the `email` field to bypass authentication entirely and log in as the administrator without knowing any password.

## Impact

- Full administrative account takeover.
- Access to all customer data, orders and administration features.
- Foothold for further attacks (data exfiltration, privilege abuse).

## Steps to reproduce

1. Browse to the login page (`/#/login`).
2. In the **Email** field enter the following payload, and put anything in the password field:

   ```
   ' OR 1=1--
   ```

3. Submit the form.

The resulting backend query resolves to something equivalent to:

```sql
SELECT * FROM Users WHERE email = '' OR 1=1--' AND password = '...'
```

The `OR 1=1` makes the `WHERE` clause always true, and `--` comments out the password check. The application returns a valid JWT for the **first** user in the table — the administrator.

4. You are now authenticated as `admin@juice-sh.op`.

To target a specific account instead of the first row:

```
' OR email='admin@juice-sh.op'--
```

## Root cause

User input is inserted into the SQL string via concatenation:

```js
// vulnerable pattern
db.query("SELECT * FROM Users WHERE email = '" + req.body.email + "' AND password = '" + hash(req.body.password) + "'")
```

There is no input validation, no parameterization, and error responses further help the attacker refine payloads.

## Remediation

- **Use parameterized queries / prepared statements.** Never concatenate user input into SQL.

  ```js
  db.query("SELECT * FROM Users WHERE email = ? AND password = ?", [email, hash(password)])
  ```

- Use an ORM with safe query building (e.g. Sequelize `where` clauses).
- Apply input validation (RFC-compliant email format).
- Enforce least-privilege database accounts.
- Return generic authentication errors to avoid leaking query structure.

## References

- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
- [OWASP WSTG — Testing for SQL Injection](https://owasp.org/www-project-web-security-testing-guide/)
- [CWE-89](https://cwe.mitre.org/data/definitions/89.html)
