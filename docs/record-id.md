# Record ID, seal and signature

AttestScope Protocol · version 0.1.1 · normative for every implementation, including Riftveil.

The key words MUST, MUST NOT, SHOULD and MAY are used as in RFC 2119.

## 1. Format

```
<ISSUER>-<YYMMDD>-<HHMMSS>-<RRRR>-<RRRR>-<RRRR>
RV-261026-143205-7K3M-Q9XD-2B8F
```

| Part | Rule |
|---|---|
| `ISSUER` | 2 to 6 letters A to Z, registered in [`ISSUERS.md`](../ISSUERS.md). `RV` is Riftveil. An implementation MUST NOT use a prefix it has not registered. `XT` is reserved for tests and MUST NOT appear in a record used for a decision. |
| `YYMMDD` | UTC date at which the ID is assigned (years 2000 to 2099). |
| `HHMMSS` | UTC time at which the ID is assigned, to the second, from a clock synchronised with a public time source. |
| `RRRR-RRRR-RRRR` | 60 bits from a cryptographically secure random generator, written in Crockford base32 (`0-9 A-Z` without `I L O U`), 12 characters in 3 groups of 4. |

Pattern: `^[A-Z]{2,6}-\d{6}-\d{6}-[0-9A-HJKMNP-TV-Z]{4}-[0-9A-HJKMNP-TV-Z]{4}-[0-9A-HJKMNP-TV-Z]{4}$`

## 2. Assignment

1. The ID MUST be assigned by code, at the moment the record is created.
2. The ID MUST NOT be produced, suggested or completed by a language model. Models repeat "random" values; an ID written by a model is not unique.
3. The ID MUST NOT be derived from the content (a hash of the project or of the report): two identical projects assessed twice are two records.
4. An ID MUST NOT be reused, even for a new version of the same record. A new version gets a new ID and points to the previous one.
5. Readers MUST accept lower case and spaces; the canonical form is upper case with hyphens.

## 3. Why duplicates are practically excluded

- Two issuers cannot collide: the prefixes differ.
- Within one issuer, two IDs can only collide if they are assigned in the same second **and** draw the same 60 random bits (1 chance in 1.15 × 10¹⁸ for any pair).
- At 1 million records a day for one issuer, the probability of a single duplicate is about 1 in 500 million per year. At 1,000 records per second, about 1 in 70,000 per year.
- This is a probability, not a mathematical impossibility. An issuer that needs absolute uniqueness MAY also check each new ID against its own register and draw again on a match; it MUST NOT shorten the random part to do so.

## 4. Seal

An issuer MUST seal each record of version 0.1.1, so that it can confirm, without storing anything, that it issued the ID for this content. (In version 0.1 the seal was recommended.)

- **Fingerprint.** SHA-256, in hexadecimal, of the content the issuer defines as the record content. The definition (profile) MUST be published and MUST include the ID whenever the issuer produced the content itself.
- **Message.** `"<label>|" + id + "|" + fingerprint`, in UTF-8. For a follow-up: `"<label>|" + id + "|" + fingerprint + "|" + sealed_at`.
- **Seal.** `S<k>-` followed by the first 160 bits of `HMAC-SHA256(key_k, message)`, in Crockford base32, 8 groups of 4. `k` is the key number; a new key gets a new number and old keys stay available for verification.
- **One label per origin.** An issuer that both produces reports and registers reports produced elsewhere MUST use a different label for each, so that a seal can never be presented as the other origin. The follow-up has its own label.
- **Key.** It MUST stay secret with the issuer, in a secret store, never in code, a shared file or a conversation.
- **Verification endpoint.** The issuer MUST publish one that answers valid or not valid, and the origin, for `{ id, fingerprint, seal }`, and stores nothing.

A valid seal shows that the issuer assigned the ID, at the time given by its clock, for this fingerprint and this origin. A seal concerns date and integrity; the substance stays with the person who signs.

**Legal scope.** A seal defined in this section is a code computed by the issuer with a secret key. It shows that the issuer assigned the ID, at the time given by the issuer's clock, for this fingerprint and this origin.

## 5. Signature

Since version 0.1.1, the issuer also signs each message it seals, with a public-key signature. A seal can only be checked by the issuer; a signature can be checked by anyone, with the issuer's public key, without asking the issuer.

- **Algorithm.** Ed25519 (RFC 8032), pure, without pre-hashing.
- **Message.** Exactly the message of the seal (section 4), byte for byte, in UTF-8. The signature therefore covers the same ID, fingerprint, origin and, for a follow-up, seal time.
- **Encoding.** The 64-byte signature in base64url (RFC 4648, section 5), without padding: 86 characters. Only the canonical encoding is valid: a verifier MUST reject a value whose last character carries non-zero spare bits. Neither the signature nor the seal may be used as an identifier.
- **Field.** `proof.signature` in the record, `follow_up.proof.signature` in the follow-up: `{ "alg": "ed25519", "key": "<key id>", "value": "<base64url>", "keys": "<URL of the key list>" }`. `keys` is informative: anyone can write a file, so a verifier MUST NOT use it to choose the key list (section 6).
- **Required.** A 0.1.1 file MUST carry `proof.signature`, and a sealed follow-up of a 0.1.1 file MUST carry `follow_up.proof.signature`. An issuer that has no signing key writes version 0.1 files.
- **Private key.** It MUST be created and kept by the issuer in a secret store, never in code, a shared file or a conversation. It MUST NOT be the sealing key. It MUST be available only to the production service of the issuer, never to test or preview deployments reachable by others; an issuer SHOULD also bind signing to its production host name.
- **Signing later (countersignature).** An issuer MAY sign, at a later time, the message of a report whose seal it has just checked, only when the record is dated before the issuer started signing (`valid_from` of its first key). This lets a 0.1 record become a 0.1.1 record when a follow-up is added. A verifier MUST show such a signature as made after the date of the record. An issuer MUST NOT sign later a report dated after it started signing: such a record already carries its signature, so a stolen sealing key cannot be turned into new signatures for it.

A valid signature shows that the holder of the issuer's private key signed this message. It says nothing more than the seal, and nothing less: it concerns date and integrity, never the substance.

## 6. Public keys

The issuer MUST publish its public keys in a key list, at a fixed address registered with its prefix in [`ISSUERS.md`](../ISSUERS.md), and SHOULD publish the same list in at least two other places it does not host on the same server. A verifier MUST take the key list from the address registered for the prefix of the `record_id` (or from a copy of that list it already trusts), never from the record file. Riftveil: `https://riftveil.ai/keys.json`, `keys/riftveil.json` in the AttestScope specification repository on GitHub, and `https://attestscope.ai/keys/riftveil.json`.

```json
{
  "issuer": "RV",
  "updated": "2026-10-26",
  "rules": "…",
  "keys": [
    {
      "id": "ed25519-1",
      "alg": "ed25519",
      "public_key": "…43 base64url characters…",
      "status": "active",
      "valid_from": "2026-10-26",
      "valid_until": null,
      "compromised_at": null
    }
  ]
}
```

| Field | Rule |
|---|---|
| `id` | `ed25519-<n>`, numbered like the sealing keys. A number is never reused. |
| `public_key` | The 32-byte public key, base64url without padding. |
| `status` | `active` (signs today), `retired` (no longer signs; its signatures stay valid) or `compromised`. |
| `valid_from`, `valid_until` | Dates (UTC) between which the key signed. `valid_until` is `null` while the key is active. |
| `compromised_at` | For a compromised key: the date from which its private part may have been known to someone else. |
| `issuer`, `updated`, `rules` | The issuer prefix, the date of the last change, and the rules of the list in plain words. Informative. |

Rules of the list:

1. No key is ever removed from the list. A retired key stays, with its dates.
2. A compromised key stays on the list, marked `compromised` and dated. A verifier MUST NOT report a signature made with it as valid on its own, whatever its date; the record then needs other evidence (a time stamp, a register).
5. A verifier MUST show when the time of a record (the time in its ID, or `sealed_at` for a follow-up) is before `valid_from` of the key that signed it (a signature made later, see section 5), and MUST NOT report as valid a signature over a record dated after `valid_until`.
6. A verifier MUST NOT accept, as a valid record of this issuer, an unsigned file (version 0.1) whose ID is dated after the issuer started signing: its signature has been removed.
3. The three published copies MUST be identical. A verifier that finds a difference MUST NOT rely on the list.
4. The issuer keeps checking its HMAC seals with every sealing key for at least twelve months after announcing that it will stop checking them.

## 7. Time stamps

`proof.timestamps` (and `follow_up.proof.timestamps`) is an OPTIONAL list. Each entry is a time stamp over the SHA-256 of the same message as the seal and the signature:

```json
"timestamps": [
  { "type": "rfc3161", "tsa": "Name of the authority", "imprint": "…64 hex…", "token": "…base64 of the DER TimeStampToken…" }
]
```

| Field | Rule |
|---|---|
| `type` | `rfc3161`. Other types MAY be defined later. |
| `tsa` | Name of the time-stamping authority, informative. |
| `imprint` | SHA-256, in hexadecimal, of the message (section 4). It MUST equal the message imprint inside the token. |
| `token` | The RFC 3161 `TimeStampToken`, DER, in base64. |

A time stamp proves the date independently of the issuer. A time stamp issued by a qualified trust service provider under a qualified time-stamping service is a qualified electronic time stamp within the meaning of Regulation (EU) No 910/2014. An organisation MAY use its own time-stamping authority. Several time stamps MAY be added to the same record. At Riftveil, qualified time stamps are planned as an option with the licences; Riftveil adds none today. The date and time of the issuer's clock, sealed and signed (`issued_at` through the ID, `sealed_at`), are the issuer's time stamp (at Riftveil: the Riftveil time stamp).

## 8. Riftveil profiles

| Origin | Label | Profile | Fingerprint input |
|---|---|---|---|
| generated on riftveil.ai | `riftveil-seal-v1` | `riftveil-report-v1` | `JSON.stringify({ v: 1, id, protocol, model, project, context, report })` |
| produced elsewhere, registered | `riftveil-register-v1` | `riftveil-content-v1` | `JSON.stringify({ v: 1, assistant, project, context, report })` |
| follow-up and decision (0.1.1) | `riftveil-decision-v1` | `riftveil-decision-v2` | see [record-file.md](record-file.md#follow-up-and-decision-follow_up-optional) |
| follow-up and decision (0.1) | `riftveil-decision-v1` | `riftveil-decision-v1` | see [record-file.md](record-file.md#follow-up-and-decision-follow_up-optional) |

The label of the follow-up stays `riftveil-decision-v1` for both profiles: the profile version is inside the fingerprint input (`v`). Strings are used exactly as sent to and returned by the model (generated) or as pasted by the person who registers (registered). Verification: `POST https://riftveil.ai/api/verify`. Keys: `https://riftveil.ai/keys.json`.

## 9. Record file

The record file is the record itself, in JSON: see [record-file.md](record-file.md). A record is verified on its record file, never on a printout, a screenshot or a PDF alone: the ID, the fingerprint, the seal and the signature can be copied onto an altered document, the text of a record file cannot be altered without the check failing. See [verification.md](verification.md).
