# AttestScope

**AttestScope — Verifiable Human Accountability for AI-Assisted Reasoning.** AttestScope is a human accountability record: accountability in the sense of human responsibility, who answers for what, not in the legal sense of a text. This repository holds the AttestScope Protocol, the open specification of the record of human judgment behind a signature, when a project was written with AI.

An AttestScope record holds what was assessed, the report of the second reader, what the person who decides checked, who owns the open questions, the decision, the document that went out, and a proof that none of it has changed: fingerprints, the issuer's seals and the issuer's Ed25519 signatures, which anyone can check with the published public key. It proves what was reviewed and when. Never that the decision was right.

**Version 0.1.1.** Publisher and reference implementation: [Riftveil](https://riftveil.ai). Website: https://attestscope.ai

## Contents

| Path | What |
|---|---|
| [`docs/record-file.md`](docs/record-file.md) | The record file: fields, the follow-up and decision block, the final document, completing a follow-up, the two forms of a record |
| [`docs/record-id.md`](docs/record-id.md) | Record ID, fingerprint profiles, seals, signatures, public keys and time stamps (normative) |
| [`docs/verification.md`](docs/verification.md) | How to verify a record, endpoints, widget, known limits |
| [`docs/conformance.md`](docs/conformance.md) | Requirements for writers, issuers and verifiers |
| [`docs/security.md`](docs/security.md) | Security and privacy considerations, threat model |
| [`docs/implement.md`](docs/implement.md) | Implementer guide, code in JavaScript and Python |
| [`docs/test-vectors.json`](docs/test-vectors.json) | Test vectors (public test keys, prefix `XT`) |
| [`docs/changelog.md`](docs/changelog.md) | Versions, compatibility rule, governance |
| [`schema/attestscope-0.1.1.schema.json`](schema/attestscope-0.1.1.schema.json) | JSON Schema (draft 2020-12) of the record file, version 0.1.1 |
| [`schema/assay-record-0.1.schema.json`](schema/assay-record-0.1.schema.json) | JSON Schema of version 0.1 files, still valid |
| [`keys/riftveil.json`](keys/riftveil.json) | Riftveil's public keys, identical to https://riftveil.ai/keys.json |
| [`examples/elena-lambert.json`](examples/elena-lambert.json) | Illustrative record, signed with the public test keys (prefix `XT`: never valid for a decision); [`test-keys.json`](test-keys.json) is its key list |
| [`examples/0.1/`](examples/0.1/) | The three illustrative records of version 0.1 |
| [`ISSUERS.md`](ISSUERS.md) | Registered issuer prefixes |

## In one minute

1. A tool writes the record file: the texts, a fingerprint (SHA-256) computed from them, the issuer's seal on that fingerprint, and the issuer's signature on the same message.
2. The person who decides adds the follow-up (who checked, what, who owns each question, the decision, a signature, and if they wish the fingerprint of the document that goes out). It has its own fingerprint, seal and signature. It can be completed later: the new follow-up keeps the earlier one.
3. Anyone verifies the file: recompute the fingerprints from the texts, check the signatures with the issuer's public key, and ask the issuer about the seals. Only fingerprints are sent; the issuer stores nothing.

Try it: https://riftveil.ai/verify.html

## Comments and changes

Comments: open an issue, or write to hello@riftveil.ai. A new issuer prefix is requested by email to hello@riftveil.ai (see [`ISSUERS.md`](ISSUERS.md)). The source of the website attestscope.ai is kept in a separate repository.

## Licence

Specification: CC BY-ND 4.0 (see [`LICENSE-SPEC.md`](LICENSE-SPEC.md)). Schema, examples and code: Apache 2.0 (see [`LICENSE`](LICENSE)).
