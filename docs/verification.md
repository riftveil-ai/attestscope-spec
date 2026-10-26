# Verification

## The rule

A record is verified on its **record file**, never on a printout, a screenshot or a PDF alone.

1. Check the form: `issued_at` repeats the time of the `record_id`, the ID is not dated in the future, every text field is a string. Otherwise: **not conforming**. Recompute the fingerprint from the content of the record file, following its profile.
2. If it differs from the fingerprint written in the file: **content altered**. Stop.
3. **Signature** (version 0.1.1). Take the issuer's key list from the address registered for the prefix of the `record_id` ([`ISSUERS.md`](../ISSUERS.md); Riftveil: `https://riftveil.ai/keys.json`), never from `proof.signature.keys`, which is written in the file. Find the key named by `proof.signature.key`, and check the Ed25519 signature over the seal message, built from the recomputed fingerprint and the origin the file states. Not valid, missing, or made with a key absent from the list: **signature not valid**. Made with a key marked `compromised`: **key compromised**, not valid on its own. Made with a key whose `valid_from` is after the date of the record: valid, shown as **signed later** (section 5 of [Record ID, seal and signature](record-id.md#5-signature)). A version 0.1 file (no signature) whose ID is dated after the issuer started signing: **signature removed**, not valid. This step needs nobody's answer: the issuer is not asked.
4. **Seal.** Send `{ id, fingerprint (recomputed), seal }` to the issuer's endpoint. If the seal is not valid: **not issued**. If it is valid for another origin than the one the file states: **forged origin**. Stop.
5. Otherwise: **valid**. Show the content of the file and ask the person to compare it with the document they were given. If they differ, the document was altered, whatever it says.
6. If the file holds a `follow_up`: recompute its fingerprint (it includes the report fingerprint recomputed in step 1). Different: **follow-up altered**. No seal: **follow-up present, not sealed**. Otherwise check its signature as in step 3, over the follow-up message (with `sealed_at`), and send it with the request of step 4 as `decision: { fingerprint, seal, sealedAt }`; the answer adds `decision: { valid, sealedAt }`.
7. If the follow-up holds `previous`: verify each earlier follow-up the same way, down the chain, including its signature when it is a 0.1.1 follow-up. An earlier follow-up that is altered, not valid, badly signed, or not sealed strictly before the one that completes it is reported as such; the follow-up that completes it stays valid, since its fingerprint covers only the earlier proof. A verifier MAY stop after 20 earlier follow-ups and report the file as not conforming.
8. If the follow-up holds `final_document`: when a person presents a document, compute the SHA-256 of its bytes on their device and compare it with `final_document.sha256`. Equal: **this is the document that was decided**. Different: **not the document that was decided**.

Steps 3 and 4 answer the same question in two ways: the signature with the public key, by anyone; the seal by the issuer. A verifier without network access to the issuer can still check the signature, with a copy of the key list.

Checking the three values alone (ID, fingerprint, seal) only proves that they belong together, not that a given text is the record.

## Riftveil endpoints

| Endpoint | Body | Answer |
|---|---|---|
| `POST https://riftveil.ai/api/verify` | `{ id, fingerprint, seal, signature?, decision? }` | `{ valid: true, id, issuedAt, origin, signature?, decision? }` or `{ valid: false }`; `signature` is `{ valid, key, status?, reason? }`, and a key marked compromised always gives `valid: false` |
| `POST https://riftveil.ai/api/register` | `{ fingerprint, profile: "riftveil-content-v1" }` | `{ id, seal, signature, issuedAt, origin: "registered", profile }` |
| `POST https://riftveil.ai/api/decide` | `{ id, report_fingerprint, report_seal, fingerprint }` | `{ seal, sealedAt, origin, signature, report_signature }` |
| `GET https://riftveil.ai/keys.json` | | The key list (section 6 of [Record ID, seal and signature](record-id.md#6-public-keys)) |

`/api/verify` accepts requests from any site (CORS), receives fingerprints only and stores nothing; given `signature: { key, value }` (and `decision.signature`), it also checks the signatures with the published keys, as a convenience. `/api/register` and `/api/decide` are used from riftveil.ai pages, behind a bot check; they also receive fingerprints only and keep nothing but a daily counter per hashed IP address. `report_signature` is the signature of the report's seal message: it lets a 0.1 record, or a registered one, be completed into a 0.1.1 record. `https://riftveil.ai/issuers.json` lists the registered prefixes.

## Pages and widget

- `https://riftveil.ai/verify.html` (FR: `/fr/verify.html`): verification by record file, or by the three values with a warning. The signature is checked in the browser with the public key; a final document can be compared there; "Complete the follow-up" reopens the record to add a follow-up that completes the earlier one.
- `https://riftveil.ai/register.html` (FR: `/fr/register.html`): registration of a report produced elsewhere; the fingerprint is computed in the browser and the record file is downloaded; the follow-up can then be filled in and sealed, as for a report generated on riftveil.ai.
- Widget for any page (it verifies the assessment, its signature, and the seal and signature of the follow-up; earlier follow-ups and the follow-up itself are shown on riftveil.ai/verify, which is the full verifier):

```html
<div data-riftveil-verify data-lang="fr"></div>
<script src="https://riftveil.ai/embed/verify.js" defer></script>
```

The host page's Content-Security-Policy must allow `script-src` and `connect-src https://riftveil.ai`.

## What remains possible, and what to do about it

| Risk | Status |
|---|---|
| Altered text in a PDF | Detected when the record file is used; not detected on a PDF alone. A qualified electronic seal on the PDF is planned. |
| Document changed after the decision | Detected when the follow-up names a final document and the document is presented for comparison. |
| ID copied from another record | Detected: the record file shows the other record's text. |
| Registered report presented as generated | Detected: one label per origin, in the seal and in the signature. |
| Fabricated project assessed for real | Not detectable, and not the purpose: the seal proves provenance and integrity, never truth. |
| Theft of the sealing key | Key in a secret store, rotation by key number without invalidating earlier seals. The signing key is a separate key. |
| Theft of the signing key | Key in a secret store, numbered; the key is marked compromised and dated in the published list, and is never removed from it. |
| Issuer no longer available | The signature stays verifiable with the published key list. |
| Follow-up added by someone other than the person named | Not prevented: the seal proves when, not who. The name is shown as declared, never as verified. |
| Signature removed to pass as 0.1 | Detected: an unsigned file dated after the issuer started signing is rejected. |
| Nobody verifies | Organisational: make the record file mandatory in the decision file. |
