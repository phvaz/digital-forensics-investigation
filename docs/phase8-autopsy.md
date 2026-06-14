# Phase 8 — Visual Analysis with Autopsy

## Context

Autopsy is a graphical digital forensics platform built on top of The Sleuth Kit.
Everything analyzed manually in Phases 4 through 7 — partition layout, file listing,
deleted file recovery, metadata extraction, timeline — Autopsy performs automatically
through its ingest module system, presenting results in an organized, navigable interface.

This phase serves as the consolidation point of the entire investigation: all artifacts
found individually in previous phases appear here correlated in a single environment,
confirming the consistency of the findings and demonstrating that manual CLI analysis
and graphical analysis produce identical results.

## Environment

This phase was performed on the host Windows machine using **Autopsy 4.23.1**,
pointing to the `evidence.dd` file located in the VirtualBox shared folder.

## What Was Done

A new case was created in Autopsy with `evidence.dd` as the data source. All available
ingest modules were enabled. After processing was complete, results were reviewed across
all artifact categories: deleted files, file metadata, and the visual timeline. An HTML
report was generated and exported as a permanent record of the analysis.

## Procedure

### 1. Create new case
```
Autopsy → New Case
Case Name: evidence
Examiner: Paulo Vaz
```

### 2. Add data source
```
Add Data Source → Disk Image or VM File
Path: evidences/evidence.dd
Timezone: America/Sao_Paulo
```

### 3. Ingest modules enabled
All available modules were enabled, including:
- Recent Activity
- Hash Lookup
- File Type Identification
- Extension Mismatch Detector
- Embedded File Extractor
- Picture Analyzer
- Keyword Search
- Email Parser
- Encryption Detection
- PhotoRec Carver
- Data Source Integrity

### 4. Results explored

**Deleted Files → All (2)**
Autopsy identified the same two orphan inodes found by `fls -r` in Phase 4:
- `OrphanFile-13` — inode 13, deleted at 13:33:09 BRT
- `OrphanFile-14` — inode 14, deleted at 13:33:09 BRT

Both flagged as `Unallocated` with `unknown` hash lookup status — consistent
with the zeroed blocks observed in Phase 4 via `istat`.

**Images (1) → File Metadata tab**
`foto_reuniao.png` was identified at inode 15, status `Allocated`, 442 bytes.
The File Metadata panel displayed the full `istat` output internally — confirming
that Autopsy calls TSK under the hood for every file inspection. MD5 and SHA256
hashes were automatically calculated and displayed.

**Timeline → Details view**
The visual timeline showed all file system events organized by file path:
- `/lost+found (4)` — filesystem initialization events
- `/ (4)` — root directory events
- `$OrphanFiles (8)` — 8 events across the two deleted files (MACB × 2)
- `/foto_reuniao.png (4)` — 4 MACB events
- `/log_sistema.log (4)` — 4 MACB events

This visual representation is the graphical equivalent of the `mactime` output
produced in Phase 7 — the same events, the same timestamps, rendered as an
interactive chart.

### 5. Report generated
```
Generate Report → HTML Report
```
Report saved to: `laudo/autopsy_report/report.html`

## Key Concepts

**Autopsy is TSK with a GUI**
Every result displayed in Autopsy corresponds directly to a TSK command executed
internally. The File Metadata panel runs `istat`. The Deleted Files view runs
`fls -d`. The Timeline runs `mactime`. Understanding the CLI tools from Phases 4
and 7 means understanding exactly what Autopsy does — the GUI adds navigation
and correlation, not new analysis capabilities.

**Ingest modules automate what we did manually**
The modules that ran automatically in this phase correspond directly to manual
steps in previous phases:

| Autopsy module | Manual equivalent | Phase |
|---|---|---|
| File Type Identification | `file` / `foremost` signatures | Phase 5 |
| Picture Analyzer | `exiftool` | Phase 6 |
| Hash Lookup | `md5sum` / `sha256sum` | Phase 3 |
| PhotoRec Carver | `foremost` | Phase 5 |
| Data Source Integrity | `ewfverify` | Phase 3 |

**Why run both CLI and Autopsy?**
CLI tools produce auditable, line-by-line output that can be directly referenced
in a forensic report. Autopsy provides navigation, correlation, and visualization
that would be impractical in a terminal for large cases. In professional practice,
both are used: CLI for documentation, Autopsy for analysis and reporting.

**Cross-phase confirmation**
Every finding in this phase confirmed findings from previous phases:
- Inodes 13 and 14 (Phase 4) → `OrphanFile-13` and `OrphanFile-14` (Autopsy)
- Deletion timestamp 12:33:09 (Phase 7) → confirmed in Deleted Files timestamps
- `foto_reuniao.png` at inode 15 (Phase 4) → confirmed in Images view
- 442 bytes PNG (Phase 5 carving) → confirmed in File Metadata

## Screenshots

- `screenshots/phase8_deleted_files.png` — deleted files view showing OrphanFile-13 and 14
- `screenshots/phase8_file_metadata.png` — file metadata panel with istat output
- `screenshots/phase8_timeline.png` — timeline counts view
- `screenshots/phase8_timeline_details.png` — timeline details view by file path
- `screenshots/phase8_report_cover1.png` — HTML report cover page (case info)
- `screenshots/phase8_report_cover2.png` — HTML report ingest history and modules

## Report

The full Autopsy HTML report is available at:
[`laudo/autopsy_report/report.html`](../laudo/autopsy_report/report.html)
