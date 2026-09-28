# Verification

## The rule

A record is verified on its **proof file**, never on a printout, a screenshot or a PDF alone.

1. Recompute the fingerprint from the content of the proof file, following its profile.
2. If it differs from the fingerprint written in the file: **content altered**. Stop.
3. Send `{ id, fingerprint (recomputed), seal }` to the issuer's endpoint.
4. If the seal is not valid: **not issued**. If it is valid for another origin than the one the file states: **forged origin**. Stop.
5. Otherwise: **valid**. Show the content of the file and ask the person to compare it with the document they were given. If they differ, the document was altered, whatever it says.
6. If the file holds a `follow_up`: recompute its fingerprint (it includes the report fingerprint recomputed in step 1). Different: **follow-up altered**. No seal: **follow-up present, not sealed**. Otherwise send it with the request of step 3 as `decision: { fingerprint, seal, sealedAt }`; the answer adds `decision: { valid, sealedAt }`.

Checking the three values alone (ID, fingerprint, seal) only proves that they belong together, not that a given text is the record.

## Riftveil endpoints

| Endpoint | Body | Answer |
|---|---|---|
| `POST https://riftveil.ai/api/verify` | `{ id, fingerprint, seal, decision? }` | `{ valid: true, id, issuedAt, origin, decision? }` or `{ valid: false }` |
| `POST https://riftveil.ai/api/register` | `{ fingerprint, profile: "riftveil-content-v1" }` | `{ id, seal, issuedAt, origin: "registered", profile }` |
| `POST https://riftveil.ai/api/decide` | `{ id, report_fingerprint, report_seal, fingerprint }` | `{ seal, sealedAt }` |

`/api/verify` accepts requests from any site (CORS), receives fingerprints only and stores nothing. `/api/register` and `/api/decide` are used from riftveil.ai pages, behind a bot check; they also receive fingerprints only and keep nothing but a daily counter per hashed IP address. `https://riftveil.ai/issuers.json` lists the registered prefixes.

## Pages and widget

- `https://riftveil.ai/verify.html` (FR: `/fr/verify.html`): verification by proof file, or by the three values with a warning.
- `https://riftveil.ai/register.html` (FR: `/fr/register.html`): registration of a report produced elsewhere; the fingerprint is computed in the browser and the proof file is downloaded.
- Widget for any page (it verifies the assessment; the follow-up is shown on riftveil.ai/verify):

```html
<div data-riftveil-verify data-lang="fr"></div>
<script src="https://riftveil.ai/embed/verify.js" defer></script>
```

The host page's Content-Security-Policy must allow `script-src` and `connect-src https://riftveil.ai`.

## What remains possible, and what to do about it

| Risk | Status |
|---|---|
| Altered text in a PDF | Detected when the proof file is used; not detected on a PDF alone. A qualified electronic seal on the PDF is planned. |
| ID copied from another record | Detected: the proof file shows the other record's text. |
| Registered report presented as generated | Detected: one label per origin. |
| Fabricated project assessed for real | Not detectable, and not the purpose: the seal proves provenance and integrity, never truth. |
| Theft of the sealing key | Key in a secret store, rotation by key number without invalidating earlier seals. |
| Nobody verifies | Organisational: make the proof file mandatory in the decision file. |
