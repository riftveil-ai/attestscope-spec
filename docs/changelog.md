# Versions

| Version | Date | Changes |
|---|---|---|
| 0.1.1 | 2026-10-26 | Issuer signature: Ed25519 over the same message as each seal, in `proof.signature` and `follow_up.proof.signature`, required in 0.1.1 files; public key list (`keys.json`) and its rules; verification without the issuer. Optional `proof.timestamps` (RFC 3161 time stamps, a list). Follow-up profile `riftveil-decision-v2`: optional `final_document` (fingerprint of the document that was decided, computed on the signer's device) and `previous` (completing a sealed follow-up later, earlier follow-ups kept with their proof). Follow-up of a registered report. `format` becomes `"attestscope"`, `version` `"0.1.1"`; schema `attestscope-0.1.1.schema.json`, with a conditional rule on the fields each fingerprint profile needs. `decided_at` stated as informative. Single illustrative example (Elena Lambert). Security rules from a review before publication: later signature (countersignature) only for records dated before the issuer started signing, and shown as such; `issued_at` equal to the time of the ID; unsigned files dated after signing started rejected; canonical signature encoding; signing key kept out of test and preview deployments; the name in a follow-up is declared, never verified. No existing profile changes: 0.1 files stay valid. |
| 0.1 | 2026-09-27 | First public version: proof file, record ID, profiles `riftveil-report-v1`, `riftveil-content-v1`, `riftveil-decision-v1`, HMAC seals, follow-up with decision, verification rules, JSON Schema, test vectors. "Record file" replaces "proof file" in the text; field names are unchanged. Assay Record was the working name of the test version. In record files, the `format` field reads `assay-record` and the `version` field `0.1-draft`. |

Reports issued on riftveil.ai before 24 October 2026 carry the working numbering 1.3.x of the Riftveil protocol in `assessment.protocol` (for example `1.3.3`); their seal stays verifiable.

## Planned for 0.2

- Canonical JSON (RFC 8785) as a fingerprint input, alongside the current profiles.
- Register entry: the record without `content`, and a public log.
- Per-finding follow-up, `outcome`, and `readers[]` with their own fingerprints.
- Identity of the person who decides: the organisation's account, or a qualified electronic signature over the follow-up.

## Compatibility rule

A new version never changes how an existing profile is computed. A verifier for 0.1.1 verifies every 0.1 record file; a verifier for 0.2 will verify every 0.1 and 0.1.1 record file.

## Governance

Riftveil publishes the format and is its reference implementation. Comments go to hello@riftveil.ai. Changes are published in this table before they apply.
