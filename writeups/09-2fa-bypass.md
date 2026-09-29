# 09 — Complete Two-Factor Authentication Bypass

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1** | 9.1 (`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N`) |
| **Category** | Identification & Authentication Failures (OWASP A07:2021) |
| **Endpoint** | `POST /rest/2fa/verify` |

## Summary

The two-factor authentication (2FA) flow can be bypassed because the second-factor check is not properly bound to the authentication session, and the verification code lacks rate limiting. An attacker who has (or bypasses) the first factor can defeat 2FA either by brute-forcing the 6-digit TOTP code or by manipulating the flow so the second step is never enforced.

## Impact

- 2FA provides no additional protection — accounts are only as strong as the first factor.
- Account takeover of users who believed they were protected by 2FA.

## Steps to reproduce

### Brute force (missing rate limiting)

1. Complete the first authentication step for the target account to obtain the intermediate 2FA challenge token.
2. Send verification requests iterating all codes `000000`–`999999`:

   ```http
   POST /rest/2fa/verify HTTP/1.1
   Content-Type: application/json

   { "tmpToken": "<challenge>", "totpToken": "000000" }
   ```

3. Automate with Burp Intruder / a script. Because there is no lockout or delay, one of the 1,000,000 attempts succeeds within minutes and returns a full session token.

### Flow manipulation

1. Observe that after step one the server already issues a usable token, or that the `/verify` step can be skipped by directly requesting a protected resource with the intermediate token.
2. Access protected functionality without ever submitting a valid TOTP code.

## Root cause

- No rate limiting / account lockout on the TOTP verification endpoint.
- The second factor is not cryptographically bound to a single-use, short-lived challenge.
- Authorization decisions accept the intermediate (pre-2FA) token.

## Remediation

- **Rate-limit and lock out** repeated failed 2FA attempts (e.g. 5 tries, then temporary lockout).
- Bind the TOTP verification to a **single-use, short-expiry** challenge tied to the session.
- Ensure protected resources require the **post-2FA** token, never the intermediate one.
- Log and alert on repeated failed 2FA attempts.

## References

- [OWASP — Multifactor Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Multifactor_Authentication_Cheat_Sheet.html)
- [OWASP — Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)
- [CWE-307](https://cwe.mitre.org/data/definitions/307.html)
