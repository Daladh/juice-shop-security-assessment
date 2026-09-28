# 05 — NoSQL Injection: Denial of Service

| | |
|---|---|
| **Severity** | 🟠 High |
| **CVSS 3.1** | 6.5 (`AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:N/A:H`) |
| **Category** | Injection (OWASP A03:2021) |
| **Endpoint** | `POST /rest/products/reviews` |

## Summary

The product review update endpoint passes user input into a NoSQL (MongoDB-style) query without sanitization. By injecting a query operator with a long-running JavaScript expression (`$where` / `$ne`), an attacker can force the database to evaluate an expensive operation across the collection, exhausting resources and causing a denial of service. The same injection class also allows authentication/authorization bypasses via operators like `$ne` and `$gt`.

## Impact

- Denial of service: the endpoint hangs and consumes CPU, degrading availability for all users.
- Broader NoSQL injection can bypass filters and access/modify unintended documents.

## Steps to reproduce

1. Authenticate and capture a review-update request in Burp Suite.
2. Replace the JSON body with an injected operator that triggers heavy evaluation:

   ```json
   {
     "id": { "$ne": -1 },
     "message": "sleep",
     "$where": "function(){ var d=new Date(); while(new Date()-d < 5000){} return true; }"
   }
   ```

3. Send the request. The server blocks while the `$where` function busy-loops for each evaluated document.
4. Send several concurrent requests to amplify the effect and observe response times climbing / the service becoming unresponsive.

### Filter-bypass variant

```json
{ "id": { "$ne": null } }
```

`$ne: null` matches every document, bypassing the intended single-record filter.

## Root cause

Request JSON is passed straight into the query object. Because MongoDB interprets nested objects like `{ "$ne": ... }` as operators, attacker-controlled keys change the query's meaning. `$where` additionally evaluates arbitrary JavaScript server-side.

## Remediation

- **Reject query operators in user input** — sanitize keys beginning with `$` or containing `.` (e.g. `express-mongo-sanitize`).
- Cast and validate every field to its expected type (`id` must be an integer).
- **Disable server-side JavaScript** in the database (`--noscripting` / `security.javascriptEnabled: false`).
- Enforce query timeouts and rate limiting.

## References

- [OWASP: Testing for NoSQL Injection](https://owasp.org/www-project-web-security-testing-guide/)
- [OWASP NoSQL Injection](https://owasp.org/www-community/Injection_Flaws)
- [CWE-943](https://cwe.mitre.org/data/definitions/943.html)
