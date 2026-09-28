# Versions

| Version | Date | Changes |
|---|---|---|
| 0.1-draft | 2026-09-27 | First public draft: proof file, record ID, profiles `riftveil-report-v1`, `riftveil-content-v1`, `riftveil-decision-v1`, HMAC seals, follow-up with decision, verification rules, JSON Schema, test vectors. |

## Planned for 0.2

- Canonical JSON (RFC 8785) as a fingerprint input, alongside the current profiles.
- Timestamps from a trusted time-stamping authority (RFC 3161) in `proof.timestamp`.
- Register entry: the record without `content`.
- Per-finding follow-up, `outcome`, and `readers[]` with their own fingerprints.

## Compatibility rule

A new draft never changes how an existing profile is computed. A verifier for 0.2 verifies every 0.1 proof file.

## Governance

Human Frontier stewards the format; Riftveil is the reference implementation. While the draft is open, comments go to hello@riftveil.ai. Changes are published in this table before they apply.
