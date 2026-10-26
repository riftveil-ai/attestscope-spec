# Record file

AttestScope Protocol · version 0.1.1. One JSON file per record, UTF-8, named `<Issuer name> <record_id>.json` (Riftveil: `Riftveil RV-261026-143205-7K3M-Q9XD-2B8F.json`).

```json
{
  "format": "attestscope",
  "version": "0.1.1",
  "record_id": "RV-261026-143205-7K3M-Q9XD-2B8F",
  "issuer": "RV",
  "origin": "generated",
  "issued_at": "2026-10-26T14:32:05Z",
  "assessment": { "protocol": "1.0.0", "model": "claude-sonnet-5", "assistant": "riftveil.ai", "language": "en" },
  "content": { "project": "…", "context": "…", "report": "…" },
  "proof": {
    "fingerprint": { "alg": "sha-256", "profile": "riftveil-report-v1", "value": "…64 hex…" },
    "seal": { "value": "S1-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX", "verify": "https://riftveil.ai/api/verify" },
    "signature": { "alg": "ed25519", "key": "ed25519-1", "value": "…86 base64url characters…", "keys": "https://riftveil.ai/keys.json" }
  },
  "notice": "…"
}
```

| Field | Rule |
|---|---|
| `format`, `version` | `"attestscope"` and `"0.1.1"`. Files of version 0.1 read `"assay-record"` and `"0.1-draft"`; they stay valid and every verifier MUST still read them. |
| `origin` | `generated` (the issuer produced the report) or `registered` (produced elsewhere, only its fingerprint was sent). Covered by the seal and the signature through their label. |
| `assessment` | For `riftveil-report-v1`, `protocol` and `model` are in the fingerprint. For `riftveil-content-v1`, `assistant` is. `language` is informative. |
| `content` | The exact texts. They stay with the person who holds the file; the issuer keeps nothing. |
| `proof.fingerprint` | Recomputed by every verifier from `content` (and the fields above). Never trusted as written. |
| `proof.seal` | Checked by the issuer's endpoint against the recomputed fingerprint. |
| `proof.signature` | REQUIRED in version 0.1.1. The issuer's Ed25519 signature over the same message as the seal. Anyone can check it with the issuer's published public key, without asking the issuer. See [Record ID, seal and signature](record-id.md#5-signature). |
| `proof.timestamps` | OPTIONAL. A list of time stamps over the same message, from time-stamping authorities (RFC 3161). See [Record ID, seal and signature](record-id.md#7-time-stamps). |
| `notice` | Informative. Not covered by any fingerprint. |

## Two forms of the same record

| Form | Holds | Kept by | Used for |
|---|---|---|---|
| Record file | Everything above, `content` included | The signer, with the PDF, in the decision file | Verifying that a text is intact |
| Register entry (planned, 0.2) | Everything except `content` | A register: Microsoft 365, the client's servers, or hosted (planned) | Counting, following up, auditing the structure |

Version 0.1.1 defines the record file; the register entry is planned for 0.2.

A register never needs the project: the fingerprint lets it prove which content a record belongs to without holding it. Whoever holds the record file holds the content: share it as you would share the report.

## Follow-up and decision (`follow_up`, optional)

Filled in by the person who decides, never by a model, after the report. It has its own fingerprint, its own seal and its own signature, so it can be added without touching the proof of the assessment.

```json
"follow_up": {
  "profile": "riftveil-decision-v2",
  "verified_by": { "last_name": "…", "first_name": "…", "role": "…" },
  "checked": "What the person checked themselves",
  "questions": [ { "n": 1, "question": "…", "suggested_owner": "CFO", "assigned_to": "…" } ],
  "decision": "proceed | conditions | stop",
  "conditions": "…",
  "place": "…",
  "signature": { "method": "drawn | typed", "typed": "…", "image": "data:image/png;base64,…" },
  "final_document": { "name": "Board memo.pdf", "size": 284113, "sha256": "…64 hex…" },
  "previous": { "profile": "riftveil-decision-v2", "…": "the follow-up this one completes, with its proof" },
  "decided_at": "2026-10-26T16:05:12Z",
  "proof": {
    "fingerprint": { "alg": "sha-256", "profile": "riftveil-decision-v2", "value": "…64 hex…" },
    "seal": { "value": "S1-…", "sealed_at": "2026-10-26T16:05:12Z", "verify": "https://riftveil.ai/api/verify" },
    "signature": { "alg": "ed25519", "key": "ed25519-1", "value": "…", "keys": "https://riftveil.ai/keys.json" }
  }
}
```

| Field | Rule |
|---|---|
| `profile` | `riftveil-decision-v2` in a 0.1.1 file. `riftveil-decision-v1` (version 0.1) appears in a 0.1.1 file only under `previous`, when a 0.1 file was completed. |
| Fingerprint input | `riftveil-decision-v2`: `JSON.stringify({ v: 2, id, report_fingerprint, verified_by, checked, questions, decision, conditions, place, signature, final_document, previous })`, with every field present: empty strings when not filled, `final_document` as `{ name: "", size: 0, sha256: "" }` when absent, `previous` as `{ fingerprint, seal, sealed_at }` of the follow-up it completes, or three empty strings. `report_fingerprint` is the recomputed fingerprint of the assessment: a follow-up belongs to one report only. `riftveil-decision-v1`: the same object without `final_document` and `previous`, with `v: 1`. |
| `signature` | A simple electronic signature made on screen by the person who decides. Not to be confused with `proof.signature`, the issuer's signature. The seal and the issuer's signature prove when the follow-up existed and that it has not changed, never who wrote it. |
| `final_document` | OPTIONAL. The document that actually goes out after the decision (a memo, a contract, a deck): its file name, its size in bytes and the SHA-256 of its bytes, computed on the signer's device. The document itself is never sent. A verifier can then say whether a given document is the one that was decided. |
| `previous` | OPTIONAL. When a person completes a sealed follow-up later (answers to the questions, conditions met), the new follow-up holds the earlier one in full, with its proof, and its fingerprint covers the earlier fingerprint, seal and seal time. Each earlier follow-up keeps its own proof. The earlier follow-up MUST be sealed, and sealed strictly before the one that completes it. A file SHOULD NOT hold more than 20 earlier follow-ups. |
| `decided_at` | Informative, set by the signer's device. Not covered by the fingerprint or the seal: the proven time is `proof.seal.sealed_at`. |
| `proof.seal` | Issued by the issuer on the follow-up fingerprint and the issuer's clock (`sealed_at`); it requires a valid seal of the assessment. Only fingerprints are sent: names, decision, signature and documents never leave the signer's device. |
| `proof.signature` | REQUIRED when the follow-up of a 0.1.1 file is sealed. The issuer's Ed25519 signature over the same message as the follow-up seal. |
| Unsealed follow-up | Allowed. A verifier shows it as present but unsealed: nothing proves when it was written or that it was not changed. |

Later versions will add `outcome` (what became of the decision) and `readers[]` (the chain of second readers), each with its own fingerprint.
