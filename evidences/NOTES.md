# Evidence Files — Notes

The forensic image files are not tracked in this repository due to GitHub's
100MB file size limit:

| File | Size | Reason |
|---|---|---|
| `evidence.dd` | 200 MB | Exceeds GitHub limit |
| `evidence_e01.E01` | 201 MB | Exceeds GitHub limit |
| `analysis/unallocated.raw` | 181 MB | Exceeds GitHub limit |

These files are listed in `.gitignore` and must be generated locally by
following the procedures documented in:

- [`docs/phase2-evidence-creation.md`](../docs/phase2-evidence-creation.md) — creates `evidence.dd`
- [`docs/phase3-forensic-acquisition.md`](../docs/phase3-forensic-acquisition.md) — creates `evidence_e01.E01` and `evidence.md5`
- [`docs/phase4-tsk-analysis.md`](../docs/phase4-tsk-analysis.md) — creates `analysis/unallocated.raw`

## Integrity Verification

`evidence.md5` contains the MD5 and SHA256 hashes of the original `evidence.dd`
image generated during this investigation. These values serve as the chain of
custody record and can be used to verify the integrity of any locally reproduced
image against the original:

```
MD5:    5b98c6bdd1c195a11c8e2d6958dab25c
SHA256: d8d0d4b942f4dfd433189ae64aa4058599b5587a3018b210eb6c5bcb11d6edee
```

```bash
# Verify your locally generated image matches the original
md5sum -c evidence.md5
sha256sum -c evidence.md5
```
