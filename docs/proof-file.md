# Proof file

Assay Record · draft 0.1. One JSON file per record, UTF-8, named `<Issuer name> <record_id>.json` (Riftveil: `Riftveil RV-260927-143205-7K3M-Q9XD-2B8F.json`).

```json
{
  "format": "assay-record",
  "version": "0.1-draft",
  "record_id": "RV-260927-143205-7K3M-Q9XD-2B8F",
  "issuer": "RV",
  "origin": "generated",
  "issued_at": "2026-09-27T14:32:05Z",
  "assessment": { "protocol": "1.3.3", "model": "claude-sonnet-5", "assistant": "riftveil.ai", "language": "fr" },
  "content": { "project": "…", "context": "…", "report": "…" },
  "proof": {
    "fingerprint": { "alg": "sha-256", "profile": "riftveil-report-v1", "value": "…64 hex…" },
    "seal": { "value": "S1-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX-XXXX", "verify": "https://riftveil.ai/api/verify" }
  },
  "notice": "…"
}
```

| Field | Rule |
|---|---|
| `origin` | `generated` (the issuer produced the report) or `registered` (produced elsewhere, only its fingerprint was sent). Covered by the seal through its label. |
| `assessment` | For `riftveil-report-v1`, `protocol` and `model` are in the fingerprint. For `riftveil-content-v1`, `assistant` is. `language` is informative. |
| `content` | The exact texts. They stay with the person who holds the file; the issuer keeps nothing. |
| `proof.fingerprint` | Recomputed by every verifier from `content` (and the fields above). Never trusted as written. |
| `proof.seal` | Checked by the issuer's endpoint against the recomputed fingerprint. |

## Two forms of the same record

| Form | Holds | Kept by | Used for |
|---|---|---|---|
| Proof file | Everything above, `content` included | The signer, with the PDF, in the decision file | Verifying that a text is intact |
| Register entry | Everything except `content` | A register (Microsoft 365, the client's servers, or hosted) | Counting, following up, auditing the structure |

A register never needs the project: the fingerprint lets it prove which content a record belongs to without holding it. Whoever holds the proof file holds the content: share it as you would share the report.

## Follow-up and decision (`follow_up`, optional)

Filled in by the person who decides, never by a model, after the report. It has its own fingerprint and its own seal, so it can be added without touching the seal of the assessment.

```json
"follow_up": {
  "profile": "riftveil-decision-v1",
  "verified_by": { "last_name": "…", "first_name": "…", "role": "…" },
  "checked": "What the person checked themselves",
  "questions": [ { "n": 1, "question": "…", "suggested_owner": "CFO", "assigned_to": "…" } ],
  "decision": "proceed | conditions | stop",
  "conditions": "…",
  "place": "…",
  "signature": { "method": "drawn | typed", "typed": "…", "image": "data:image/png;base64,…" },
  "decided_at": "2026-09-27T15:16:39Z",
  "proof": {
    "fingerprint": { "alg": "sha-256", "profile": "riftveil-decision-v1", "value": "…64 hex…" },
    "seal": { "value": "S1-…", "sealed_at": "2026-09-27T15:16:39Z", "verify": "https://riftveil.ai/api/verify" }
  }
}
```

| Field | Rule |
|---|---|
| Fingerprint input | `JSON.stringify({ v: 1, id, report_fingerprint, verified_by, checked, questions, decision, conditions, place, signature })`, with every field present (empty strings when not filled). `report_fingerprint` is the recomputed fingerprint of the assessment: a follow-up belongs to one report only. |
| `proof.seal` | Issued by the issuer on the follow-up fingerprint and the issuer's clock (`sealed_at`); it requires a valid seal of the assessment. Only fingerprints are sent: names, decision and signature never leave the signer's device. |
| Unsealed follow-up | Allowed. A verifier shows it as present but unsealed: nothing proves when it was written or that it was not changed. |
| Signature | A simple electronic signature made on screen. The seal proves when the follow-up existed and that it has not changed, never who wrote it. |

Later versions of this draft add `outcome` (what became of the decision) and `readers[]` (the chain of second readers), each with its own fingerprint.
