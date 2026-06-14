# Phase 7 — Forensic Timeline

## Context

A forensic timeline is the chronological reconstruction of all recorded events
on a file system, based on the timestamps stored in each inode. It answers the
fundamental investigative question: what happened, in what order, and when?

The timeline is built from a bodyfile — a structured text format that aggregates
MACB timestamps for every file system object — then sorted and formatted using
`mactime`. This phase connects everything found in the previous phases into a
single, coherent sequence of events.

## What Was Done

A bodyfile was generated from the forensic image using `fls -m`, capturing MACB
timestamps for all files including deleted ones. The bodyfile was then processed
by `mactime` to produce a sorted, human-readable chronological timeline.

## Commands Executed

```bash
# Generate bodyfile with MACB timestamps for all files
fls -m / -r -o 2048 evidences/evidence.dd > timeline/bodyfile.txt

# Generate chronological timeline from bodyfile
mactime -b timeline/bodyfile.txt > timeline/timeline.txt

# View the timeline
cat timeline/timeline.txt
```

> Note: `mactime` produced deprecation warnings about old Perl package syntax.
> These are cosmetic warnings from the script itself and do not affect the output.

## Findings

```
Sun Jun 07 2026 12:12:15    12288 macb d/drwx------ 0  0  11   /lost+found
Sun Jun 07 2026 12:26:08        0 .a.b -/rrw-r--r-- 0  0  13   /$OrphanFiles/OrphanFile-13 (deleted)
Sun Jun 07 2026 12:27:03        0 .a.b -/rrw-r--r-- 0  0  14   /$OrphanFiles/OrphanFile-14 (deleted)
Sun Jun 07 2026 12:29:38      442 macb r/rrw-r--r-- 0  0  15   /foto_reuniao.png
Sun Jun 07 2026 12:32:00      233 macb r/rrw-r--r-- 0  0  16   /log_sistema.log
Sun Jun 07 2026 12:33:09        0 m.c. -/rrw-r--r-- 0  0  13   /$OrphanFiles/OrphanFile-13 (deleted)
                                0 m.c. -/rrw-r--r-- 0  0  14   /$OrphanFiles/OrphanFile-14 (deleted)
```

## Timeline Reconstruction

| Time | Event | Forensic significance |
|---|---|---|
| 12:12:15 | `lost+found` created | ext4 filesystem initialized — mkfs.ext4 from Phase 2 |
| 12:26:08 | OrphanFile-13 created and accessed | `transfer_receipt.txt` written to disk |
| 12:27:03 | OrphanFile-14 created and accessed | `credentials.txt` written to disk |
| 12:29:38 | `foto_reuniao.png` — full macb | Image file created and active |
| 12:32:00 | `log_sistema.log` — full macb | System log generated automatically |
| 12:33:09 | OrphanFile-13 and OrphanFile-14 — m.c. | **Both files deleted simultaneously** |

The timeline tells a clear story: two sensitive files were created between 12:26
and 12:27, and deleted 7 minutes later at 12:33 — consistent with a deliberate
attempt to destroy evidence. This sequence was reconstructed entirely from file
system metadata, independent of file content.

## Key Concepts

**MACB timestamps**
Each file system inode stores four timestamps recorded independently by the OS:

| Letter | Name | Records when... |
|---|---|---|
| M | Modified | File content was last written |
| A | Accessed | File was last opened or read |
| C | Changed | Inode metadata was last changed |
| B | Born | File was created |

In the timeline output, a `.` means that timestamp did not change at that moment.
For example, `m.c.` at 12:33:09 means only Modified and Changed timestamps were
updated — consistent with deletion, which updates inode metadata but not access time.

**The bodyfile format**
The bodyfile is an intermediate format defined by TSK. Each line represents one
file system object and contains all four timestamps alongside file path, inode
number, size, and permissions. `mactime` reads this file and sorts all events
chronologically across all files simultaneously — producing a unified view of
everything that happened on the file system.

**How this connects to previous phases**
- The inodes 13 and 14 found here are the same orphan inodes discovered in Phase 4
  via `fls -r`
- The deletion timestamp `12:33:09` matches the `Deleted` field seen in `istat`
  output from Phase 4
- The `foto_reuniao.png` at inode 15 is the same file recovered by Foremost in
  Phase 5 and analyzed by ExifTool in Phase 6 — its `Modify Date: 2026:06:07
  16:29:38 UTC` corresponds to `12:29:38 EDT` in the timeline (UTC-4)

**Timestamp tampering detection**
A timeline with manipulated timestamps would show anomalies such as:
- Access time before creation time
- Modification time earlier than birth time
- Deletion timestamp before the file was created

None of these anomalies were found in this investigation — the timeline is
internally consistent and corroborates all findings from previous phases.

## Screenshots

- `screenshots/phase7_bodyfile.png` — bodyfile generation command
- `screenshots/phase7_timeline.png` — full chronological timeline output
