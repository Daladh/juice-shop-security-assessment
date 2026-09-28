# 04 — SSRF via Profile Image URL

| | |
|---|---|
| **Severity** | 🟠 High |
| **CVSS 3.1** | 8.6 (`AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:N/A:N`) |
| **Category** | Server-Side Request Forgery (OWASP A10:2021) |
| **Endpoint** | `POST /api/Users/{id}/profileImage` (image URL) |

## Summary

The "set profile image from URL" feature makes the server fetch a user-supplied URL without validating its destination. An attacker can point it at internal addresses, causing the server to make requests on their behalf — a classic Server-Side Request Forgery. This can expose internal-only services, cloud metadata endpoints, and be used to map the internal network.

## Impact

- Access to internal services not reachable from the internet.
- Disclosure of cloud metadata / credentials (e.g. `169.254.169.254`).
- Internal port scanning and service enumeration.

## Steps to reproduce

1. Log in and open the profile page (`/#/profile`).
2. Use the **Image URL** field, or send the request directly:

   ```http
   POST /api/Users/1/profileImage HTTP/1.1
   Host: target
   Content-Type: application/json
   Authorization: Bearer <token>

   { "imageUrl": "http://169.254.169.254/latest/meta-data/" }
   ```

3. The server fetches the URL. Observe the response/behaviour or side effects to confirm the internal request was made.

Additional test targets to demonstrate reachability:

```
http://localhost:3000/
http://127.0.0.1:22/
http://internal-service.local/
```

## Root cause

The server treats the supplied URL as trusted and issues an outbound HTTP request with no allowlist, no scheme restriction, and no protection against internal / link-local address ranges.

## Remediation

- **Allowlist** permitted domains/schemes for outbound fetches.
- Block requests to private, loopback and link-local ranges (`127.0.0.0/8`, `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`, `169.254.0.0/16`).
- Resolve the hostname and validate the resulting IP **before** connecting (prevent DNS rebinding).
- Disable unneeded URL schemes (`file://`, `gopher://`, etc.).
- Where possible, have users upload the image directly instead of fetching a URL.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [OWASP SSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Server_Side_Request_Forgery_Prevention_Cheat_Sheet.html)
- [CWE-918](https://cwe.mitre.org/data/definitions/918.html)
