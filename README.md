# Parallax Commitments

Public, append-only record of predictions issued by Parallax, a single-operator
geopolitical early-warning system. Each prediction is committed here at the
moment it is issued, before its outcome is known.

## What a commitment proves
- The prediction's content existed, unchanged, by the time it was committed.
- It was issued by Parallax (every commit is GPG-signed with the key below).
- The time is independently anchored via OpenTimestamps (Bitcoin), so it
  cannot be backdated — including by the operator.

## What it does not prove
That the prediction was correct. Accuracy is scored separately against
adjudicated outcomes. This repo only guarantees the predictions weren't
written or edited after the fact.

## How to verify a prediction

### 1. Rebuild the canonical form

A commitment is taken over a closed set of exactly seven fields. Nothing else
about the prediction is committed — not its prose, not the sensor readings
behind it.

| Field | Type | Format |
|---|---|---|
| `event_class` | string | the prediction type |
| `generator_name` | string | which generator issued it |
| `horizon_days` | integer | prediction window in whole days, > 0 |
| `issued_at` | string | UTC ISO-8601, **truncated to whole seconds**, explicit `+00:00` |
| `probability` | number | rounded to 4 decimal places |
| `region` | string | theatre the prediction applies to |
| `system_version` | string | the Parallax version that issued it |

Serialise them as JSON with **keys sorted alphabetically**, **no whitespace**,
and no trailing newline — `json.dumps(obj, sort_keys=True, separators=(",", ":"))`:

```json
{"event_class":"...","generator_name":"...","horizon_days":1,"issued_at":"2026-09-21T05:06:17+00:00","probability":0.5,"region":"...","system_version":"2.5.0"}
```

Two formatting rules matter because they decide the hash:

- `issued_at` is truncated to whole seconds. Sub-second precision is dropped
  deliberately: the same instant can come back from the database with or
  without trailing zeros, and a hash that depends on that is unverifiable.
- `probability` is rounded to 4 decimal places and emitted as a JSON number.

The field set is closed and versioned. Adding a field would change the hash of
every prediction already issued, so a future addition bumps the canonical
version rather than editing this one; old predictions keep verifying under the
rules they were issued under.

### 2. Hash it and compare

SHA-256 the UTF-8 bytes of that JSON, lowercase hex. The result is the
prediction's commitment hash, and it is also the filename:

```
predictions/<sha256>.json        the canonical JSON — exactly the bytes hashed
predictions/<sha256>.json.ots    the OpenTimestamps proof for that file
```

One commit per prediction, message `commit prediction <sha256>`. So the file
name, the file contents and the commit message all carry the same hash, and
any disagreement between them is visible without trusting anything.

```sh
sha256sum predictions/<sha256>.json     # must equal the filename
```

### 3. Verify the commit signature

```sh
git verify-commit <sha>
```

with the key below. Every commit in this repository is signed, including the
first.

### 4. Verify the timestamp

```sh
ots verify predictions/<sha256>.json.ots
```

(Fresh stamps are calendar receipts; they upgrade to full Bitcoin proofs
within hours. Run `ots upgrade` on the proof to fetch the Bitcoin attestation
once it has confirmed.)

The signature proves authorship and the git history proves ordering, but both
are controlled by the operator. **The OpenTimestamps proof is the only part of
this record that the operator cannot forge**, which is why a commitment is not
considered complete without one.

## What is deliberately absent

No prediction is ever removed or amended here. A prediction that turns out
wrong stays exactly as committed — that is the point of the record. If a
commitment appears with no corresponding entry in the accuracy ledger, or a
ledger entry appears with no commitment here, treat both as unscoreable rather
than assuming which one is right.

## Signing key

Fingerprint: `7F661EFA7936D8563B668610D025FCBD4CFAEC2C`

```
-----BEGIN PGP PUBLIC KEY BLOCK-----

mDMEarDLXxYJKwYBBAHaRw8BAQdAu67qaM9jGI/a5xEncLJ9XWWFCMd9VfghqXjb
PdnOaAu0K1BhcmFsbGF4IENvbW1pdG1lbnRzIDxoby5yaWNoYXJkQGdtYWlsLmNv
bT6IkAQTFggAOBYhBH9mHvp5NthWO2aGENAl/L1M+uwsBQJqsMtfAhsDBQsJCAcC
BhUKCQgLAgQWAgMBAh4BAheAAAoJENAl/L1M+uwsmCcBALiknrms2gy5ugBVgxz4
3rjgdK07XLos4pKvTJmO7H/ZAQD2ga/u3Mx/aXiyBsz3L1FtdIQwdFNdZdGrFAzV
5rxxCQ==
=jo8C
-----END PGP PUBLIC KEY BLOCK-----
```
