# Phase 3 — Forensic Acquisition

## Context

Forensic acquisition is the most critical phase of any digital investigation.
The goal is to produce an exact, verifiable copy of the original evidence — a
forensic image — without altering a single bit of the source. Any modification
after this point can invalidate the evidence in court.

This phase produces two outputs: a raw `.dd` image and an `.E01` image, along
with cryptographic hashes that guarantee their integrity throughout the process.

## What Was Done

Hash values (MD5 and SHA256) were calculated for the raw image immediately after
creation and verified with `md5sum -c` and `sha256sum -c`. The image was then
converted to the E01 format using `ewfacquire` with case metadata. The E01 was
verified with `ewfverify -d sha256`. All three files were confirmed locally.

## Commands Executed

```bash
# Generate integrity hashes (chain of custody)
md5sum evidences/evidence.dd > evidences/evidence.md5
sha256sum evidences/evidence.dd >> evidences/evidence.md5
cat evidences/evidence.md5

# Verify hashes
md5sum -c evidences/evidence.md5
sha256sum -c evidences/evidence.md5

# Convert to E01 format with case metadata
ewfacquire evidences/evidence.dd -t evidences/evidence_e01

# Verify E01 integrity after conversion
ewfverify -d sha256 evidences/evidence_e01.E01

# Confirm all files generated
ls -lh evidences/
```

## Findings

Hash values generated:

| Algorithm | Hash |
|---|---|
| MD5 | `5b98c6bdd1c195a11c8e2d6958dab25c` |
| SHA256 | `d8d0d4b942f4dfd433189ae64aa4058599b5587a3018b210eb6c5bcb11d6edee` |

Both `md5sum -c` and `sha256sum -c` returned **OK**.

> Note: Each tool produced a `WARNING: 1 line is improperly formatted` — expected
> behavior, as `md5sum` and `sha256sum` each recognize only their own format and
> flag the other line. The OK confirmation is what matters.

E01 acquisition parameters used:
- Case number: 001
- Description: Digital Forensics Lab
- Examiner: Paulo Vaz
- Media type: fixed disk
- EWF format: EnCase 6 (.E01)
- Compression: deflate / none

### Files Generated in `evidences/`

| File | Size | Status in repository |
|---|---|---|
| `evidence.dd` | 200 MB | **Not tracked** — exceeds GitHub's 100MB limit |
| `evidence_e01.E01` | 201 MB | **Not tracked** — exceeds GitHub's 100MB limit |
| `evidence.md5` | 144 B | Tracked — contains MD5 and SHA256 hashes |

`evidence.dd` and `evidence_e01.E01` are excluded via `.gitignore` and are not
present in this repository. This was a deliberate decision: GitHub enforces a
100MB file size limit and these forensic images cannot be split without losing
their forensic validity.

The `evidence.md5` file is tracked and serves as the permanent chain of custody
record — anyone reproducing this investigation locally can verify their generated
image against the original hashes.

For reproduction instructions and hash verification, see
[`evidences/NOTES.md`](../evidences/NOTES.md).

## Key Concepts

**Chain of Custody**
The chain of custody is a documented, unbroken record of who handled the evidence,
when, and what was done with it. Cryptographic hashes are the technical backbone
of this chain: if the MD5 or SHA256 of the image changes at any point, the evidence
has been altered and is no longer forensically sound.

**MD5 vs SHA256**
MD5 is faster and historically dominant in forensic tools, but has known collision
vulnerabilities. SHA256 is cryptographically stronger. Recording both is best
practice — it satisfies legacy tools while meeting modern security standards.

**Why two formats?**
The `.dd` (raw) format is a bit-for-bit copy of the original device, universally
compatible with every forensic tool. The `.E01` (Expert Witness Format / EnCase
Evidence File) wraps the same data with embedded case metadata (examiner name,
date, description, notes), internal CRC checksums per block, and compression.
E01 is the standard format accepted in formal forensic reports and legal proceedings.

**On physical devices (real investigation):**
```bash
# Write-blocker must be connected before this step
dd if=/dev/sdX of=evidences/evidence.dd bs=4M status=progress conv=noerror,sync
# conv=noerror,sync: continue on read errors; pad bad sectors with zeros
```

## Standards Applied

- **ISO/IEC 27037:2012** — requires acquisition to be performed without altering
  the original evidence, with integrity verification
- **RFC 3227** — recommends recording hash values immediately after acquisition
- **NIST SP 800-86** — recommends using write blockers for physical media

## Screenshots

- `screenshots/phase3_hashes.png` — MD5 and SHA256 output
- `screenshots/phase3_hash_verification.png` — md5sum -c and sha256sum -c returning OK
- `screenshots/phase3_ewfacquire.png` — E01 conversion process and parameters
- `screenshots/phase3_ewfverify.png` — integrity verification result
- `screenshots/phase3_evidence_files.png` — ls -lh confirming all three files
