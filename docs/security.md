# Security and privacy considerations

Assay Record · draft 0.1.

## What is protected

| Threat | Protection | Residual risk |
|---|---|---|
| Text altered after issue | Fingerprint recomputed from the file by every verifier | None on the proof file; a PDF shown alone proves nothing |
| ID or seal copied onto another document | Verification shows the texts sealed under that ID | A person must compare the texts with the document |
| Registered report presented as generated | One seal label per origin | None |
| Follow-up moved to another report | Its fingerprint includes the report fingerprint | None |
| Follow-up edited after sealing | Its own fingerprint and seal, with the issuer's time | None if sealed; an unsealed follow-up proves nothing |
| Seal time moved | The time is inside the sealed message | None |
| Forged seal | HMAC-SHA256 truncated to 160 bits, secret key | Only the issuer can check a seal; a timestamp by a third party is planned |
| Stolen sealing key | Secret store, numbered keys, rotation | Records sealed with a stolen key cannot be told apart; the key number allows a cut-off date |
| Mass sealing by robots | Bot check, per-IP and global daily limits | None known |

## What is not protected, by design

- The truth of the project or the quality of the review. A seal proves provenance and integrity only.
- The identity of the signer. An on-screen signature is a simple electronic signature; the seal proves when the follow-up existed, not who wrote it.
- Verification without the issuer. With HMAC seals, only the issuer can say whether a seal is valid. Public-key signatures or RFC 3161 timestamps are planned, so that a record can be checked without asking its issuer.

## Privacy

- The issuer receives fingerprints, never texts, names, decisions or signatures. A SHA-256 fingerprint cannot be turned back into the text.
- The issuer stores no texts and no list of IDs or fingerprints. Only short-lived rate-limit counters, keyed on a secret hash of the IP address, are kept.
- Whoever holds a proof file holds the texts, and the names in the follow-up. Share it as you would share the report.
- A register entry without the texts is planned, for registers that must not hold the project.
