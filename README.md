# attestz-transparency
AttestZ Omni checkpoint log

## Layout

`checkpoints/YYYY/MM/<date>.json` is the signed daily checkpoint. It is published once and
never rewritten. Its `checkpoint_hash` is SHA-384 over the canonical JSON of the record without
`signatures`, `anchors` and `checkpoint_hash`, so it covers `prev_checkpoint_hash`, the link to
the day before. Both signatures (ECDSA P-256 and ML-DSA-65, AWS KMS) are over that hash.

`checkpoints/YYYY/MM/<date>.ots` is the OpenTimestamps proof for the same checkpoint, upgraded
until it reaches a Bitcoin block. The `opentimestamps` anchor inside the `.json` is the proof as
it stood at signing time, which is always `"pending": true` because Bitcoin has not confirmed it
yet. The `.ots` file beside it is the completed proof. It is a separate file so the signed record
never has to change.

## Checking a timestamp yourself

The OpenTimestamps proof commits to SHA-256 of the raw 48 bytes of `checkpoint_hash`:

```bash
f=checkpoints/2026/09/2026-09-03
digest=$(python3 -c "import json,hashlib,sys;print(hashlib.sha256(bytes.fromhex(json.load(open(sys.argv[1]))['checkpoint_hash'])).hexdigest())" $f.json)
ots verify -d "$digest" $f.ots              # with a Bitcoin node
ots --no-bitcoin verify -d "$digest" $f.ots # without one: checks the proof is for this checkpoint
ots --no-bitcoin info $f.ots                # lists each Bitcoin block and the merkle root to compare on any block explorer
```
