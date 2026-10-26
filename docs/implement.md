# Implementing AttestScope

Everything below runs as is. The fingerprints are those of the reference implementation (riftveil.ai).

## 1. Compute a fingerprint

The fingerprint input is a JSON object serialised **without spaces, without escaping non-ASCII characters, keys in the order shown**. In JavaScript this is exactly `JSON.stringify`. In Python, use `ensure_ascii=False` and `separators=(",", ":")`. Then SHA-256 over the UTF-8 bytes, in lower-case hexadecimal.

| Profile | Input object, in this key order |
|---|---|
| `riftveil-report-v1` | `v` (1), `id`, `protocol`, `model`, `project`, `context`, `report` |
| `riftveil-content-v1` | `v` (1), `assistant`, `project`, `context`, `report` |
| `riftveil-decision-v1` | `v` (1), `id`, `report_fingerprint`, `verified_by` {`last_name`, `first_name`, `role`}, `checked`, `questions` [{`n`, `question`, `suggested_owner`, `assigned_to`}], `decision`, `conditions`, `place`, `signature` {`method`, `typed`, `image`} |
| `riftveil-decision-v2` | the fields of `riftveil-decision-v1` with `v` (2), then `final_document` {`name`, `size`, `sha256`}, `previous` {`fingerprint`, `seal`, `sealed_at`} |

JavaScript (browser or Node 18+):

```js
async function sha256hex(text) {
  const d = await crypto.subtle.digest("SHA-256", new TextEncoder().encode(text));
  return [...new Uint8Array(d)].map(b => b.toString(16).padStart(2, "0")).join("");
}

function reportInput(f) {
  return JSON.stringify({ v: 1, id: f.record_id, protocol: f.assessment.protocol, model: f.assessment.model,
    project: f.content.project, context: f.content.context, report: f.content.report });
}

const file = JSON.parse(recordFileText);
const fingerprint = await sha256hex(reportInput(file));
if (fingerprint !== file.proof.fingerprint.value) throw new Error("content altered");
```

Python 3:

```python
import hashlib, json

def canon(obj):
    return json.dumps(obj, ensure_ascii=False, separators=(",", ":"))

def report_fingerprint(f):
    a, c = f["assessment"], f["content"]
    obj = {"v": 1, "id": f["record_id"], "protocol": a["protocol"], "model": a["model"],
           "project": c["project"], "context": c["context"], "report": c["report"]}
    return hashlib.sha256(canon(obj).encode("utf-8")).hexdigest()

f = json.load(open("Riftveil RV-….json", encoding="utf-8"))
assert report_fingerprint(f) == f["proof"]["fingerprint"]["value"], "content altered"
```

## 2. Check the signature with the public key

Version 0.1.1. The signature is Ed25519 over the seal message; anyone can check it with the issuer's published key list, without asking the issuer.

JavaScript (recent browsers, Node 20+):

```js
const b64u = (s) => Uint8Array.from(atob(s.replace(/-/g, "+").replace(/_/g, "/")), (c) => c.charCodeAt(0));
const label = file.origin === "registered" ? "riftveil-register-v1" : "riftveil-seal-v1";
const message = label + "|" + file.record_id + "|" + fingerprint;   // fingerprint recomputed in step 1
const list = await (await fetch(file.proof.signature.keys)).json();
const k = list.keys.find((x) => x.id === file.proof.signature.key && x.alg === "ed25519");
if (!k || k.status === "compromised") throw new Error("key unknown or compromised");
const pub = await crypto.subtle.importKey("raw", b64u(k.public_key), { name: "Ed25519" }, false, ["verify"]);
const ok = await crypto.subtle.verify({ name: "Ed25519" }, pub, b64u(file.proof.signature.value), new TextEncoder().encode(message));
if (!ok) throw new Error("signature not valid");
```

Python 3 (`pip install cryptography`):

```python
import base64, json, urllib.request
from cryptography.hazmat.primitives.asymmetric.ed25519 import Ed25519PublicKey

def b64u(s): return base64.urlsafe_b64decode(s + "=" * (-len(s) % 4))

label = "riftveil-register-v1" if f["origin"] == "registered" else "riftveil-seal-v1"
message = label + "|" + f["record_id"] + "|" + fingerprint
sig = f["proof"]["signature"]
keys = json.load(urllib.request.urlopen(sig["keys"]))["keys"]
k = next(k for k in keys if k["id"] == sig["key"] and k["alg"] == "ed25519")
assert k["status"] != "compromised", "key compromised"
Ed25519PublicKey.from_public_bytes(b64u(k["public_key"])).verify(b64u(sig["value"]), message.encode("utf-8"))  # raises if not valid
```

For a follow-up, the message is `"riftveil-decision-v1|" + id + "|" + follow-up fingerprint + "|" + sealed_at`, and the signature is `follow_up.proof.signature`. For an earlier follow-up under `previous`, repeat with its own fields.

## 3. Check the seal with the issuer

The seal is an HMAC with the issuer's secret key: only the issuer can check it. Send fingerprints only.

```js
const res = await fetch("https://riftveil.ai/api/verify", {
  method: "POST", headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ id: file.record_id, fingerprint, seal: file.proof.seal.value })
});
const r = await res.json();   // { valid: true, id, issuedAt, origin } or { valid: false }
if (!r.valid) throw new Error("seal not valid");
if (r.origin !== file.origin) throw new Error("forged origin");
```

For a file with a follow-up, add `decision: { fingerprint, seal, sealedAt }` to the same request; the answer adds `decision: { valid, sealedAt }`.

```bash
curl -s https://riftveil.ai/api/verify -H 'Content-Type: application/json' \
  -d '{"id":"RV-…","fingerprint":"…64 hex…","seal":"S1-…"}'
```

## 4. Validate the structure

```bash
pip install jsonschema
python3 -c "import json,jsonschema; jsonschema.validate(json.load(open('record.json')), json.load(open('attestscope-0.1.1.schema.json')))"
```

Version 0.1 files validate against `assay-record-0.1.schema.json`.

## 5. Test vectors

Computed with the reference implementation. The seals use the **public test key** `assay-test-key-1`, the signatures the **public Ed25519 test key** `ed25519-1` of [test-vectors.json](test-vectors.json) (seed `SHA-256("attestscope-test-ed25519-1")`, public key `2FLWc-QnkT78GPsymtkuE5xgzIMLCHzigAAgOs6D2TY`), and the prefix `XT`, reserved for tests: they let you test a sealing or signing implementation, and are never valid at riftveil.ai. Full inputs: [test-vectors.json](test-vectors.json). The illustrative record [`examples/elena-lambert.json`](../examples/elena-lambert.json) uses the same test keys, with the key list [`test-keys.json`](../test-keys.json).

| Vector | Fingerprint | Seal with the test key |
|---|---|---|
| V1 · generated (riftveil-report-v1) | `abc5ba903a307668121b1fdd0e01ce784069189c1b255e481f65f5e54ee765fc` | `S1-2ZQR-3N0J-PRKP-8G33-Q2ZT-ZPZA-QDFN-MEPH` |
| V2 · registered (riftveil-content-v1) | `2ae571cf5ba080714e2d04e867a05b09a36f4157ddbdd17d3a1574366af0f9fd` | `S1-7CBT-7WG0-0HBP-2336-45NX-2PDS-HE57-C6DK` |
| V3 · non-ASCII text and line break | `14971ae9adf597868bd4ab39783cc64c7ebff23e38d7a4f2f80d11df9d212f55` | `S1-CW1S-3W97-PB70-MKYQ-NTD5-JFQK-P8ES-B883` |
| V4 · follow-up (riftveil-decision-v1), bound to V1 | `bfdb75b1260feca991f6f59862c3cdc7e3910f424612244e9fdeb999a951d82e` | `S1-FEK5-BRSD-MC99-S95R-2V46-C40R-CHYM-ARK2` |
| V6 · follow-up (riftveil-decision-v2), completes V4, with a final document | `2f283adb5660de3e785220c90a27a7eb9abe43fb310dffbdda579d6b2f87b817` | `S1-JZHS-ME8Y-5A25-4S9Z-7YBJ-HMYE-7J26-B9AT` |

Signatures with the Ed25519 test key: V5, over the seal message of V1, `2jKXUE5glnWjDwQg-u4d6-GiOJKflAYbj3lgjDrsqunL4gs6TOentYbyUpIbbFX7IYIZ2qGmHYvVtpVAyI9BDg`; V6, over its seal message, `kWyKO9MpoYuqXmawQUVZZH__4k3rWgWIyHo5mTNIB3w_mIiZqTLRv4sEAn4GGnbBWt5BUPn7jhNwt95aa8nTCw`.

V1 input, byte for byte:

```
{"v":1,"id":"XT-260101-000000-0000-0000-0000","protocol":"1.3.3","model":"test-model","project":"Project text.","context":"","report":"Report text."}
```

Seal: `"S1-"` + the first 160 bits of `HMAC-SHA256(key, "riftveil-seal-v1|" + id + "|" + fingerprint)`, in Crockford base32 (`0123456789ABCDEFGHJKMNPQRSTVWXYZ`), 32 characters in 8 groups of 4. For V1 the HMAC input is `riftveil-seal-v1|XT-260101-000000-0000-0000-0000|abc5ba903a307668121b1fdd0e01ce784069189c1b255e481f65f5e54ee765fc`.

## 6. Add verification to a page

```html
<div data-riftveil-verify data-lang="en"></div>
<script src="https://riftveil.ai/embed/verify.js" defer></script>
```

The host page's Content-Security-Policy must allow `script-src` and `connect-src https://riftveil.ai`.

## 7. Become an issuer

Register a prefix in [ISSUERS](#issuers) (prefix, name, contact, verification endpoint, address of the key list), publish your fingerprint profiles if you add any, create your signing key yourself and keep it, like your sealing key, in a secret store, publish your key list, and follow the issuer rules of [Conformance](#conformance).
