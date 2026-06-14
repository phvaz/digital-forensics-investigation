# Phase 5 — File Carving and Recovery

## Context

File carving is a recovery technique that operates at the raw binary level,
reconstructing files from disk data without relying on the file system structure.
While TSK (Phase 4) recovers files through inode references, carving works even
when inodes have been overwritten or zeroed out — making it the last resort in
file recovery.

In this investigation, `icat` returned empty in Phase 4 due to block zeroing by
the VirtualBox shared folder filesystem. Carving provides an independent recovery
path that bypasses the file system entirely.

## What Was Done

Foremost was used to scan the entire forensic image and recover files based on
their binary signatures (magic numbers). The tool searched for known file type
signatures and reconstructed any matching files found in the image.

## Commands Executed

```bash
# Run carving against the forensic image
foremost -i evidences/evidence.dd -o recovered/foremost -v

# Review recovered files
ls -lh recovered/foremost/
ls -lh recovered/foremost/png/

# Review the audit log
cat recovered/foremost/audit.txt
```

## Findings

```
Num      Name (bs=512)         Size      File Offset     Comment
0:      00020486.png          442 B        10488832       (640 x 480)

1 FILES EXTRACTED
png:= 1
```

- **1 file recovered** — the `meeting_photo.png` planted in Phase 2
- Found at byte offset **10488832** in the image
- File size: **442 bytes**, dimensions: **640 x 480** — intact and valid
- Recovered to `recovered/foremost/png/00020486.png`
- `audit.txt` generated automatically at `recovered/foremost/audit.txt`

> The filename `00020486.png` was assigned by Foremost based on the sector offset
> where the PNG signature was found. The original filename is not recoverable
> through carving — only through inode-based methods like `fls` and `icat`.

## Key Concepts

**What is a file signature (magic number)?**
Every file format reserves the first few bytes of the file for a type identifier —
called a magic number or file signature. This identifier is independent of the file
extension. Examples:

| Format | Hex signature | ASCII |
|---|---|---|
| JPEG | `FF D8 FF` | `ÿØÿ` |
| PNG | `89 50 4E 47` | `‰PNG` |
| PDF | `25 50 44 46` | `%PDF` |
| ZIP | `50 4B 03 04` | `PK..` |

Foremost scans the entire image byte by byte looking for these patterns, then reads
forward until it finds the corresponding end-of-file marker, reconstructing the file.

**Carving vs. inode-based recovery**
TSK recovers files through inode references — the file system's own records.
Carving ignores the file system entirely and works directly on raw bytes. This makes
carving effective even when:
- The file system has been intentionally corrupted
- Inodes have been zeroed out (as observed in Phase 4)
- The disk has been partially reformatted

**Foremost output structure**
Foremost creates one subfolder per file type recovered (`png/`, `jpg/`, `pdf/`, etc.)
and an `audit.txt` file listing every recovered file with its byte offset in the image,
size, and dimensions. The `audit.txt` is identical to the verbose terminal output —
it is automatically saved to disk as a permanent record of the carving session.

**Carving limitation: no reliable metadata**
Files recovered through carving have no associated inode, no original filename,
and no trustworthy timestamps. The only reliable information is the file content
itself and its byte offset in the image. This distinction is important when
writing the forensic report — carved files must be described differently from
inode-recovered files.

## Screenshots

- `screenshots/phase5_foremost_running.png` — foremost execution and audit output
- `screenshots/phase5_recovered_files.png` — recovered directory structure and file listing