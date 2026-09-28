# 02 — Forged JWT: Authentication Bypass

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1** | 9.8 (`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`) |
| **Category** | Identification & Authentication Failures (OWASP A07:2021) |
| **Endpoint** | JWT-protected API routes |

## Summary

The application relies on JSON Web Tokens (JWT) for session authentication but does not validate them securely. Two independent weaknesses allow token forgery:

1. **`alg: none`** — the server accepts unsigned tokens when the algorithm header is set to `none`.
2. **Weak / disclosed signing key** — the RSA/HMAC key material is predictable or discoverable, allowing an attacker to sign a valid token for any user.

Either path lets an attacker impersonate arbitrary users, including the administrator.

## Impact

- Impersonation of any user (including `admin`) without credentials.
- Complete authentication bypass across the entire API.

## Steps to reproduce

### Path A — Unsigned token (`alg: none`)

1. Capture a valid JWT (e.g. from a normal login) using Burp Suite.
2. Decode it (`jwt.io` or `jwt_tool`). The payload looks like:

   ```json
   { "status": "success", "data": { "email": "user@juice-sh.op", "role": "customer" } }
   ```

3. Change the header algorithm to `none` and edit the payload to the admin identity:

   ```json
   { "email": "admin@juice-sh.op", "role": "admin" }
   ```

4. Re-encode with an **empty signature** (trailing dot):

   ```
   eyJhbGciOiJub25lIn0.<base64url-payload>.
   ```

5. Replace the `Authorization: Bearer` header with the forged token and replay a protected request. The server accepts it.

### Path B — Forged signed token

1. Recover the signing key (public certificate shipped with the app, or a weak HMAC secret).
2. Use `jwt_tool` to sign a modified payload:

   ```bash
   python3 jwt_tool.py <token> -X k -pk public.pem   # key confusion
   # or
   python3 jwt_tool.py <token> -S hs256 -p <weak-secret>
   ```

3. Submit the re-signed token — it validates as authentic.

## Root cause

- The JWT library is configured to accept multiple algorithms, including `none`.
- The verification step does not pin a single expected algorithm.
- Signing key material is embedded in the shipped code or is guessable.

## Remediation

- **Pin the algorithm explicitly** on verification (e.g. `RS256` only); reject `none`.
- Store signing keys as secrets, rotate them, never ship them with the client.
- Validate all standard claims (`exp`, `iat`, `iss`, `aud`).
- Prefer strong asymmetric signing (RS256/ES256) with properly protected private keys.

## References

- [Auth0 — Critical vulnerabilities in JWT libraries](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/)
- [OWASP JWT Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/JSON_Web_Token_for_Java_Cheat_Sheet.html)
- [CWE-347](https://cwe.mitre.org/data/definitions/347.html)
