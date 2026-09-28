# 06 — Broken Access Control: Viewing Other Users' Baskets (IDOR)

| | |
|---|---|
| **Severity** | 🟡 Medium |
| **CVSS 3.1** | 5.3 (`AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:N/A:N`) |
| **Category** | Broken Access Control (OWASP A01:2021) |
| **Endpoint** | `GET /rest/basket/{basketId}` |

## Summary

The basket endpoint returns basket contents based on the `basketId` in the URL, but only checks that the caller is authenticated — not that the basket **belongs to them**. By changing the `basketId`, any logged-in user can read another user's shopping basket. This is a classic Insecure Direct Object Reference (IDOR).

## Impact

- Horizontal privilege escalation: users can read other customers' basket contents.
- Disclosure of what other users are purchasing (privacy breach).
- Same missing-check pattern often extends to modifying other baskets.

## Steps to reproduce

1. Log in as a normal user. Your own basket loads at, say, `GET /rest/basket/1`.
2. Note your `basketId` in the request.
3. In Burp Suite (or the browser), change the ID and replay:

   ```http
   GET /rest/basket/2 HTTP/1.1
   Host: target
   Authorization: Bearer <your-token>
   ```

4. The server returns basket `2`'s contents — a basket that belongs to a different user — even though your token is not associated with it.
5. Iterate through IDs (`1..N`) to enumerate other users' baskets.

## Root cause

The authorization logic verifies *authentication* (a valid JWT) but omits an *ownership* check. It never confirms that `basket.UserId === currentUser.id` before returning the record.

## Remediation

- **Enforce object-level authorization** on every request: confirm the authenticated user owns (or may access) the requested object.

  ```js
  const basket = await Basket.findByPk(req.params.id)
  if (!basket || basket.UserId !== req.user.id) return res.status(403).send()
  ```

- Do not rely on client-supplied IDs for access decisions.
- Consider unpredictable identifiers (UUIDs) as defence-in-depth, but never as the primary control.
- Add automated access-control tests to CI.

## References

- [OWASP: Insecure Direct Object Reference Prevention](https://cheatsheetseries.owasp.org/cheatsheets/Insecure_Direct_Object_Reference_Prevention_Cheat_Sheet.html)
- [OWASP A01:2021 — Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
- [CWE-639](https://cwe.mitre.org/data/definitions/639.html)
