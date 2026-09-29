# 08 — Zip Slip: Arbitrary File Overwrite

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1** | 9.1 (`AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:H/A:H`) |
| **Category** | File Handling / Path Traversal (OWASP A01:2021) |
| **Endpoint** | `POST /file-upload` (archive extraction) |

## Summary

The application extracts uploaded ZIP archives without validating the entry paths inside them. A malicious archive containing an entry with a directory-traversal path (`../../`) causes the extracted file to be written **outside** the intended directory, allowing an attacker to overwrite arbitrary files on the server — configuration files, application code, or scheduled scripts — potentially leading to remote code execution.

## Impact

- Overwrite of arbitrary files the service can write to.
- Application code or config tampering → possible remote code execution.
- Integrity and availability compromise.

## Steps to reproduce

1. Craft a ZIP archive whose entry name contains a traversal path. Example with Python:

   ```python
   import zipfile
   with zipfile.ZipFile("evil.zip", "w") as z:
       z.writestr("../../../../app/config/pwned.txt", "overwritten by Zip Slip")
   ```

2. Upload `evil.zip` through the file-upload feature that performs extraction.
3. The extractor joins the base output directory with the entry path without normalization, so the `../` sequences escape the target directory and the file lands at `/app/config/pwned.txt`.
4. Confirm the file was written outside the upload folder.

## Root cause

During extraction the code does something equivalent to:

```js
const target = path.join(outputDir, entry.fileName)   // no validation
fs.writeFileSync(target, entry.data)
```

`entry.fileName` is attacker-controlled and may contain `../`, so `target` resolves outside `outputDir`.

## Remediation

- **Canonicalize and validate** each entry's destination path before writing:

  ```js
  const target = path.resolve(outputDir, entry.fileName)
  if (!target.startsWith(path.resolve(outputDir) + path.sep)) {
    throw new Error("Path traversal detected in archive entry")
  }
  ```

- Reject entries containing `..`, absolute paths, or symlinks.
- Extract into a sandboxed directory with least privilege.
- Enforce limits on entry count and size (also mitigates zip bombs).

## References

- [Snyk — Zip Slip vulnerability](https://security.snyk.io/research/zip-slip-vulnerability)
- [OWASP — Path Traversal](https://owasp.org/www-community/attacks/Path_Traversal)
- [CWE-22](https://cwe.mitre.org/data/definitions/22.html)
