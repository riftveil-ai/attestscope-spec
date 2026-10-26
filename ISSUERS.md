# Registered issuer prefixes

An implementation that writes AttestScope records needs its own prefix before its first record used for a decision. Request one by email to hello@riftveil.ai, with the prefix, name, contact, URL of the verification endpoint and URL of the key list. Each request is reviewed and may be declined, for example if the prefix could be confused with another issuer's.

| Prefix | Issuer | Verification | Keys | Since |
|---|---|---|---|---|
| `RV` | Riftveil (riftveil.ai) | `POST https://riftveil.ai/api/verify` | `https://riftveil.ai/keys.json` | 2026-09-27 |
| `XT` | Reserved for tests, never valid for a decision | none | `https://attestscope.ai/test-keys.json` (public test key) | 2026-09-27 |
