# Security and privacy considerations

AttestScope Protocol · version 0.1.1.

## What the format covers

| Risk | Countermeasure | Residual risk |
|---|---|---|
| Text altered after issue | Fingerprint recomputed from the file by every verifier | None on the record file; a PDF shown alone proves nothing |
| ID or seal copied onto another document | Verification shows the texts sealed under that ID | A person must compare the texts with the document |
| Registered report presented as generated | One label per origin, in the seal and in the signature | None |
| Follow-up moved to another report | Its fingerprint includes the report fingerprint | None |
| Follow-up edited after sealing | Its own fingerprint, seal and signature, with the issuer's time | None if sealed; an unsealed follow-up proves nothing |
| Earlier follow-up edited after it was completed | Its proof stays in the file; the follow-up that completes it covers its fingerprint, seal and seal time | None: the earlier follow-up is reported as altered |
| Document changed after the decision | `final_document.sha256`, compared on the reader's device | Only when the follow-up names the document and the document is presented |
| Seal time moved | The time is inside the sealed and signed message | None |
| Forged seal | HMAC-SHA256 truncated to 160 bits, secret key | Only the issuer can check a seal: the signature covers the same message |
| Forged signature | Ed25519, private key in a secret store | None known |
| Issuer unreachable or gone | The signature is checked with the published key list | The list must stay published; it is published in three places |
| Stolen sealing key | Secret store, numbered keys, rotation; separate signing key | Seals made with a stolen key cannot be told apart; the signature still holds |
| Stolen signing key | Secret store, numbered keys; the key is marked compromised and dated, never removed | Signatures made with it no longer prove anything on their own; a time stamp by a third party, or a register, is then needed |
| Mass sealing by robots | Bot check, per-IP and global daily limits | None known |

## What it does not cover, by design

- The truth of the project or the quality of the review. A seal or a signature proves provenance and integrity only.
- The identity of the signer. An on-screen signature is a simple electronic signature; the seal and the issuer's signature prove when the follow-up existed, not who wrote it.
- A date proven independently of the issuer. The issuer's clock is sealed and signed; a time stamp by a time-stamping authority (RFC 3161, `proof.timestamps`) proves the date without relying on the issuer.

## Privacy

- For verification, registration and the decision seal, the issuer receives fingerprints only, never texts, names, decisions, signatures or documents. An issuer that also produces the report receives the texts to write it and keeps none of them; the provider of the AI model it uses may keep them for a limited period, stated on the issuer's privacy page. A SHA-256 fingerprint cannot be turned back into the text; it only lets someone confirm a text they already have, or guess.
- The fingerprint of a final document is computed on the signer's device; the document is never sent.
- The issuer stores no texts and no list of IDs or fingerprints. Only short-lived rate-limit counters, keyed on a hash of the IP address made with a secret key of their own, are kept.
- Whoever holds a record file holds the texts, the names in the follow-up, and the name of the final document. Share it as you would share the report.
- A register entry without the texts is planned, for registers that must not hold the project.
