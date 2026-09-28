# Conformance

Assay Record · draft 0.1. The key words MUST, MUST NOT, SHOULD and MAY are used as in RFC 2119.

A conformance claim names the role and the draft version: "writes Assay Record 0.1 proof files", "verifies Assay Record 0.1 proof files", "issues Assay Record 0.1 records under prefix XX".

## Writer

A tool that writes proof files:

1. MUST produce UTF-8 JSON that validates against `schema/assay-record-0.1.schema.json`.
2. MUST copy the texts into `content` exactly as they were assessed, sent and returned, without trimming, reformatting or translation.
3. MUST compute `proof.fingerprint.value` from those exact texts with the profile it names.
4. MUST NOT let a language model write, suggest or complete the `record_id`, the fingerprint, the seal or any field of `follow_up`.
5. MUST leave `follow_up` absent, or with every field empty, until a person fills it in.
6. SHOULD name the file `<Issuer name> <record_id>.json`.

## Issuer

An issuer assigns IDs and seals:

1. MUST use a prefix registered in [ISSUERS](#issuers), and MUST NOT issue records under `XT` for real decisions.
2. MUST assign IDs by code, in the format of [Record ID and seal](#record-id-and-seal), from a clock synchronised with a public time source.
3. MUST keep its sealing keys in a secret store, number them, and keep retired keys available for verification.
4. MUST use one label per origin and one for the follow-up.
5. MUST publish a verification endpoint that answers valid or not valid, the origin, and for a follow-up the seal time, for fingerprints only, and stores nothing.
6. MUST NOT keep the texts, a list of IDs or a list of fingerprints for verification purposes.
7. SHOULD rate-limit sealing and registration, and SHOULD protect them against automated abuse.

## Verifier

A tool that verifies proof files:

1. MUST recompute every fingerprint from the file, following its profile, and MUST NOT trust a fingerprint as written.
2. MUST report "altered" when a recomputed fingerprint differs from the one in the file, and stop there for that part.
3. MUST ask the issuer's endpoint to check each seal, sending only the ID, fingerprints, seals and seal times.
4. MUST report the origin returned by the issuer, and "forged origin" when it differs from the one the file states.
5. MUST show the sealed texts so that a person can compare them with the document they were given.
6. MUST NOT present a check of the ID, fingerprint and seal alone as a verification of a document.
7. SHOULD state in its result that a valid record proves provenance and integrity, never that a decision was right.
