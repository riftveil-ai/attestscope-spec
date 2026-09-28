# Record ID and seal

Assay Record · draft 0.1 · normative for every implementation, including Riftveil.

The key words MUST, MUST NOT, SHOULD and MAY are used as in RFC 2119.

## 1. Format

```
<ISSUER>-<YYMMDD>-<HHMMSS>-<RRRR>-<RRRR>-<RRRR>
RV-260927-143205-7K3M-Q9XD-2B8F
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

## 4. Seal (recommended)

An issuer SHOULD seal each record so that anyone can check, without the issuer storing anything, that the ID was issued by it for this content.

- **Fingerprint.** SHA-256, in hexadecimal, of the content the issuer defines as the record content. The definition (profile) MUST be published and MUST include the ID whenever the issuer produced the content itself.
- **Seal.** `S<k>-` followed by the first 160 bits of `HMAC-SHA256(key_k, "<label>|" + id + "|" + fingerprint)`, in Crockford base32, 8 groups of 4. `k` is the key number; a new key gets a new number and old keys stay available for verification.
- **One label per origin.** An issuer that both produces reports and registers reports produced elsewhere MUST use a different label for each, so that a seal can never be presented as the other origin.
- **Key.** It MUST stay secret with the issuer, in a secret store, never in code, a shared file or a conversation.
- **Verification endpoint.** The issuer MUST publish one that answers valid or not valid, and the origin, for `{ id, fingerprint, seal }`, and stores nothing.
- A valid seal proves who issued the ID, when, for which fingerprint and with which origin. It never says the project is right.

### Riftveil profiles

| Origin | Label | Profile | Fingerprint input |
|---|---|---|---|
| generated on riftveil.ai | `riftveil-seal-v1` | `riftveil-report-v1` | `JSON.stringify({ v: 1, id, protocol, model, project, context, report })` |
| produced elsewhere, registered | `riftveil-register-v1` | `riftveil-content-v1` | `JSON.stringify({ v: 1, assistant, project, context, report })` |

| follow-up and decision | `riftveil-decision-v1` | `riftveil-decision-v1` | see [proof-file.md](proof-file.md#follow-up-and-decision-follow_up-optional); the HMAC input adds the seal time: `"riftveil-decision-v1|" + id + "|" + fingerprint + "|" + sealed_at` |

Strings are used exactly as sent to and returned by the model (generated) or as pasted by the person who registers (registered). Verification: `POST https://riftveil.ai/api/verify`.

## 5. Proof file

The proof file is the record itself, in JSON: see [proof-file.md](proof-file.md). A record is verified on its proof file, never on a printout, a screenshot or a PDF alone: the ID, the fingerprint and the seal can be copied onto an altered document, the text of a proof file cannot be altered without the check failing. See [verification.md](verification.md).

A timestamp from a trusted time-stamping authority (RFC 3161, qualified under eIDAS in the European Union) MAY be added in `proof.timestamp` in a later draft; it would prove the date independently of the issuer. Riftveil does not add one yet.
