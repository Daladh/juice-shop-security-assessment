# 11 — Web3: Exposed Seed Phrase & Private Key Derivation

| | |
|---|---|
| **Severity** | 🔴 Critical |
| **CVSS 3.1** | 9.1 (`AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N`) |
| **Category** | Cryptographic Failures / Sensitive Data Exposure (OWASP A02:2021) |
| **Endpoint** | Frontend bundle / Web3 wallet feature |

## Summary

The application's Web3 / blockchain feature exposes a wallet **seed phrase (mnemonic)** through client-side code and public content. Because a BIP-39 mnemonic deterministically derives the wallet's private keys (BIP-32/BIP-44), anyone who recovers the phrase gains full control of the associated wallet and its funds. Secrets that grant control of cryptographic assets must never be reachable by a client.

## Impact

- Full compromise of the wallet: an attacker can derive the private key and sign transactions.
- Theft of any funds/tokens held by the wallet.
- Impersonation of the wallet owner on-chain.

## Steps to reproduce

1. Enumerate the application's exposed content and JavaScript bundles (DevTools → Sources, or fetch the bundle directly).
2. Search the client-side code and public data (feedback, comments, exposed files) for a 12/24-word mnemonic pattern, e.g.:

   ```
   grep -Eo '([a-z]+ ){11,23}[a-z]+' bundle.js
   ```

3. Recover the candidate seed phrase.
4. Demonstrate control **without touching real funds** by deriving the address offline and confirming it matches the target wallet:

   ```
   BIP-39 mnemonic → seed → BIP-32 master key → BIP-44 account 0 → address
   ```

   Any standard offline library reproduces the same address, proving the phrase controls the wallet.

## Root cause

A secret with the highest possible sensitivity (a wallet mnemonic) was embedded in client-delivered code / public content. Client-side code is fully visible to users, so anything shipped to the browser is effectively public.

## Remediation

- **Never ship seed phrases or private keys to the client**, in any form (source, config, comments, storage).
- Keep signing keys server-side in a secrets manager or hardware wallet / HSM; sign server-side.
- If a key has ever been exposed, treat it as compromised: **rotate immediately** and move funds to a new wallet.
- Add secret-scanning to CI to block credentials and mnemonics from being committed or bundled.

## References

- [OWASP — Cryptographic Failures (A02:2021)](https://owasp.org/Top10/A02_2021-Cryptographic_Failures/)
- [OWASP Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
- [CWE-798: Use of Hard-coded Credentials](https://cwe.mitre.org/data/definitions/798.html)
