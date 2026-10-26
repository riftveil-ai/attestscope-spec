# Conformance

AttestScope Protocol · version 0.1.1. The key words MUST, MUST NOT, SHOULD and MAY are used as in RFC 2119.

A conformance claim names the role and the version: "writes AttestScope 0.1.1 record files", "verifies AttestScope 0.1.1 record files", "issues AttestScope 0.1.1 records under prefix XX". A conformance claim to this specification concerns the format only. The attestation recorded in an AttestScope Record is the signer's, on the scope of their own review.

## Writer

A tool that writes record files:

1. MUST produce UTF-8 JSON that validates against `schema/attestscope-0.1.1.schema.json`.
2. MUST copy the texts into `content` exactly as they were assessed, sent and returned, without trimming, reformatting or translation.
3. MUST compute `proof.fingerprint.value` from those exact texts with the profile it names, and fill the fields that profile needs: `assessment.protocol` and `assessment.model` for `riftveil-report-v1`, `assessment.assistant` for `riftveil-content-v1`.
4. MUST NOT let a language model write, suggest or complete the `record_id`, the fingerprint, the seal, the signature or any field of `follow_up`.
5. MUST leave `follow_up` absent, or with every field empty, until a person fills it in.
6. MUST compute the fingerprint of a final document on the signer's device, and MUST NOT send the document.
7. When a person completes a sealed follow-up, MUST keep the earlier follow-up in full under `previous`, with its proof, and MUST NOT change it.
8. SHOULD name the file `<Issuer name> <record_id>.json`.

## Issuer

An issuer assigns IDs, seals and signs:

1. MUST use a prefix registered in [ISSUERS](#issuers), and MUST NOT issue records under `XT` for real decisions.
2. MUST assign IDs by code, in the format of [Record ID, seal and signature](#record-id-seal-and-signature), from a clock synchronised with a public time source.
3. MUST keep its sealing keys and its signing keys in a secret store, as separate keys, number them, and keep retired sealing keys available for verification.
4. MUST use one label per origin and one for the follow-up.
5. MUST sign every message it seals in a 0.1.1 record, with the private key of an active key of its published list.
6. MUST publish its key list at the address named in its signatures, keep it identical wherever it is published, never remove a key from it, and mark and date a compromised key.
7. MUST publish a verification endpoint that answers valid or not valid, the origin, and for a follow-up the seal time, for fingerprints only, and stores nothing.
8. MUST NOT keep the texts, a list of IDs or a list of fingerprints for verification purposes.
9. MUST check the seal of the report before sealing a follow-up.
10. SHOULD rate-limit sealing and registration, and SHOULD protect them against automated abuse.

## Verifier

A tool that verifies record files:

1. MUST recompute every fingerprint from the file, following its profile, and MUST NOT trust a fingerprint as written.
2. MUST report "altered" when a recomputed fingerprint differs from the one in the file, and stop there for that part.
3. MUST check each signature of a 0.1.1 file with the issuer's published key list, MUST report a missing signature as not valid, and MUST NOT report a signature made with a key marked compromised as valid on its own.
4. MUST ask the issuer's endpoint to check each seal, sending only the ID, fingerprints, seals and seal times, when the issuer can be reached.
5. MUST report the origin returned by the issuer, and "forged origin" when it differs from the one the file states.
6. MUST verify each earlier follow-up under `previous` and report its result separately.
7. MUST show the sealed texts so that a person can compare them with the document they were given.
8. MUST compute the fingerprint of a presented final document on the person's device, and MUST NOT send the document.
9. MUST NOT present a check of the ID, fingerprint and seal alone as a verification of a document.
10. MUST still verify version 0.1 files, which carry no signature.
11. SHOULD state in its result that a valid record proves provenance and integrity, never that a decision was right.
