# Phase 4 — Analysis with The Sleuth Kit (TSK)

## Context

The Sleuth Kit is the forensic engine that powers Autopsy. Running TSK directly
from the command line provides full transparency over what happens beneath the
graphical interface — and is the standard approach in professional forensic labs
where every step must be auditable and reproducible.

This phase covers partition analysis, file system traversal, deleted file recovery,
inode inspection, and pattern searching directly in the binary image.

## What Was Done

The forensic image was analyzed using TSK command-line tools to identify partition
layout, list all files (including deleted), inspect inode metadata, and search for
relevant strings embedded in the binary data. File content recovery via `icat` was
attempted but returned empty due to block zeroing by the VirtualBox shared folder
— a finding documented and explained below. Content was successfully recovered
through raw string search and unallocated space extraction.

## Commands Executed

```bash
# Identify partitions and sector offsets
mmls evidences/evidence.dd > analysis/mmls_output.txt
cat analysis/mmls_output.txt

# List all active files
fls -o 2048 evidences/evidence.dd > analysis/fls_output.txt
cat analysis/fls_output.txt

# List deleted files recursively (fls -d returned no output; -r revealed orphan inodes)
fls -r -o 2048 evidences/evidence.dd >> analysis/fls_output.txt

# Inspect inode metadata for deleted files
istat -o 2048 evidences/evidence.dd 13 > analysis/istat_13.txt
istat -o 2048 evidences/evidence.dd 14 > analysis/istat_14.txt

# Attempt content recovery by inode (returned empty — blocks zeroed)
icat -o 2048 evidences/evidence.dd 13
icat -o 2048 evidences/evidence.dd 14

# Search for strings directly in the binary image
strings evidences/evidence.dd | grep -i "PIX" > analysis/strings_grep.txt
strings evidences/evidence.dd | grep -E "[0-9]{3}\.[0-9]{3}\.[0-9]{3}-[0-9]{2}" >> analysis/strings_grep.txt

# Extract unallocated space and search for deleted content
blkls -o 2048 evidences/evidence.dd > analysis/unallocated.raw
strings analysis/unallocated.raw | grep -i "admin" >> analysis/strings_grep.txt
```

## Findings

### Partition layout (`mmls`)
```
002:  000:000   0000002048   0000409599   Linux (0x83)
```
Partition starts at sector **2048** — this offset was used in all subsequent commands.

### Active files (`fls`)
```
d/d 11: lost+found
r/r 15: foto_reuniao.png
r/r 16: log_sistema.log
```

### Deleted files (`fls -r`)
```
+ -/r * 13:     OrphanFile-13
+ -/r * 14:     OrphanFile-14
```
Inodes 13 and 14 correspond to the intentionally deleted files `transfer_receipt.txt`
and `credentials.txt`. The original filenames were lost because the directory entries
were removed — TSK labels them as `OrphanFile`.

### Inode metadata (`istat`)
Both inodes returned:
```
size: 0
Direct Blocks: (empty)
Deleted: 2026-06-07 12:33:09
```
Block pointers were zeroed by the VirtualBox shared folder filesystem at deletion time.
However, **MACB timestamps were fully preserved** — creation, access, modification,
and deletion times are all recorded and constitute valid forensic evidence that the
files existed and were deliberately deleted.

### Content recovery (`icat`)
Returned empty for both inodes 13 and 14. Expected result given `size: 0` and empty
block pointers confirmed by `istat`. Content recovery was achieved through raw string
search instead (see below).

### String search (`strings + grep`)
```
Transferencia PIX R$ 5000 CPF 123.456.789-00
2026-06-07 12:24:03 TRANSFERENCIA PIX valor=5000
```
PIX transfer data and CPF number recovered directly from raw binary image bytes,
independent of any file system structure.

### Unallocated space (`blkls + strings`)
```
senha_acesso: admin@2024
```
Credential data from the deleted `credentials.txt` recovered from 181MB of
unallocated space extracted by `blkls`.

## Key Concepts

**What is an offset?**
`mmls` returns the start sector of each partition. This value (2048 in this case)
must be passed to every subsequent TSK command via `-o` so the tool knows where
the file system begins inside the image.

**What is an inode?**
An inode is a data structure that stores metadata about a file: permissions, owner,
timestamps, and the disk block pointers where the file's data is stored. When a file
is deleted, its directory entry is removed but the inode structure remains — including
timestamps — until the space is reused.

**Why did `icat` return empty?**
The VirtualBox shared folder (vboxsf) filesystem zeroes data blocks at deletion time,
unlike a native Linux ext4 filesystem where blocks are simply marked as unallocated
and data persists until overwritten. This is an environment-specific behavior — on a
real physical device, `icat` would typically recover the file content successfully.

**Deleting is not erasing**
Even with zeroed inodes, the actual file content was found in the raw binary image
via `strings + grep` and in unallocated space via `blkls`. This is the fundamental
principle of digital forensics: operating system deletion only removes references,
not data.

**MACB Timestamps**
Each inode records four timestamps:
- **M** — Modified: last time file content was changed
- **A** — Accessed: last time the file was opened or read
- **C** — Changed: last time inode metadata was modified
- **B** — Born: file creation time

These timestamps feed directly into the forensic timeline built in Phase 7.

## Screenshots

- `screenshots/phase4_mmls.png` — partition layout output
- `screenshots/phase4_fls.png` — active file listing
- `screenshots/phase4_fls_deleted.png` — deleted orphan inodes via `fls -r`
- `screenshots/phase4_istat.png` — inode metadata for inodes 13 and 14
- `screenshots/phase4_strings_pix.png` — PIX and CPF recovered from raw image
- `screenshots/phase4_strings_unallocated.png` — credential recovered from unallocated space
