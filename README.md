# Assay Record

An open format for the record of human judgment behind a signature, when a project was written with AI.

An Assay Record holds what was assessed, the report of the second reader, what the person who decides checked, who owns the open questions, the decision, and a proof that none of it has changed. It proves what was reviewed and when. Never that the decision was right.

**Status: draft 0.1.** Reference implementation: [Riftveil](https://riftveil.ai). Stewarded by Human Frontier. Website: https://assayrecord.ai

## Contents

| Path | What |
|---|---|
| [`docs/proof-file.md`](docs/proof-file.md) | The proof file: fields, the follow-up and decision block, the two forms of a record |
| [`docs/record-id.md`](docs/record-id.md) | Record ID, fingerprint profiles and seals (normative) |
| [`docs/verification.md`](docs/verification.md) | How to verify a record, endpoints, widget, known limits |
| [`docs/conformance.md`](docs/conformance.md) | Requirements for writers, issuers and verifiers |
| [`docs/security.md`](docs/security.md) | Security and privacy considerations, threat model |
| [`docs/implement.md`](docs/implement.md) | Implementer guide, code in JavaScript and Python |
| [`docs/test-vectors.json`](docs/test-vectors.json) | Test vectors (public test key, prefix `XT`) |
| [`docs/changelog.md`](docs/changelog.md) | Versions, compatibility rule, governance |
| [`schema/assay-record-0.1.schema.json`](schema/assay-record-0.1.schema.json) | JSON Schema (draft 2020-12) of the proof file |
| [`examples/`](examples/) | Three illustrative records (prefix `XT`, reserved for tests: their seals are not valid) |
| [`ISSUERS.md`](ISSUERS.md) | Registered issuer prefixes |

## In one minute

1. A tool writes the proof file: the texts, a fingerprint (SHA-256) computed from them, and the issuer's seal on that fingerprint.
2. The person who decides adds the follow-up (who checked, what, who owns each question, the decision, a signature). It has its own fingerprint and seal.
3. Anyone verifies the file: recompute the fingerprints from the texts, then ask the issuer whether the seals are valid. Only fingerprints are sent; the issuer stores nothing.

Try it: https://riftveil.ai/verify.html

## Comments and changes

Draft 0.1 is open for comment: open an issue, or write to hello@riftveil.ai. A new issuer prefix is requested by pull request on [`ISSUERS.md`](ISSUERS.md). The source of the website assayrecord.ai is kept in a separate repository.

## Licence

Specification: CC BY-ND 4.0 while in draft (see [`LICENSE-SPEC.md`](LICENSE-SPEC.md)). Schema, examples and code: Apache 2.0 (see [`LICENSE`](LICENSE)).
