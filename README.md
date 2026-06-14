# Digital Forensics Investigation — From Zero to Report

> **A complete simulation of a digital forensic investigation using open source tools, conducted in a controlled environment (Kali Linux virtual machine), focused on technical documentation, chain of custody, and forensic report production.**

---

## Table of Contents

1. [Context and Motivation](#1-context-and-motivation)
2. [Objectives](#2-objectives)
3. [Environment and Tools](#3-environment-and-tools)
4. [Project Architecture](#4-project-architecture)
5. [Project Phases](#5-project-phases)
   - [Phase 1 — Environment Setup](#phase-1--environment-setup)
   - [Phase 2 — Synthetic Evidence Creation](#phase-2--synthetic-evidence-creation)
   - [Phase 3 — Forensic Acquisition](#phase-3--forensic-acquisition)
   - [Phase 4 — Analysis with The Sleuth Kit (TSK)](#phase-4--analysis-with-the-sleuth-kit-tsk)
   - [Phase 5 — File Carving and Recovery](#phase-5--file-carving-and-recovery)
   - [Phase 6 — Metadata and EXIF Analysis](#phase-6--metadata-and-exif-analysis)
   - [Phase 7 — Forensic Timeline](#phase-7--forensic-timeline)
   - [Phase 8 — Visual Analysis with Autopsy](#phase-8--visual-analysis-with-autopsy)
   - [Phase 9 — Forensic Report Production](#phase-9--forensic-report-production)
6. [Investigation Scenario](#6-investigation-scenario)
7. [Repository Structure](#7-repository-structure)
8. [Technical and Regulatory References](#8-technical-and-regulatory-references)
9. [About the Author](#9-about-the-author)

---

## 1. Context and Motivation

Digital forensics is a discipline that demands methodological rigor, technical mastery of tools, and a deep understanding of the file systems under investigation. In real cases, a single mistake during evidence collection can render the entire legal process invalid.

This project was designed as a complete practical laboratory, simulating a full investigation from start to finish: from creating a forensic image to issuing a technical report. The choice of open source tools — TSK, Autopsy, ExifTool, Foremost — reflects the standard used by professional forensic laboratories and security agencies worldwide.

The virtualized environment (Kali Linux in a VM) ensures complete isolation from the host system, replicating the operational discipline required in real contexts where evidence integrity is legally relevant.

---

## 2. Objectives

- Demonstrate the complete workflow of a digital forensic investigation, from setup to report
- Apply in practice the concepts of chain of custody, evidence integrity, and forensic methodology
- Explore open source tools widely adopted in the industry (TSK, Autopsy, ExifTool, Foremost)
- Correlate forensic artifacts to reconstruct the incident timeline
- Produce professional-grade technical documentation replicable in real environments

---

## 3. Environment and Tools

### Environment

| Component | Specification |
|---|---|
| Operating System | Kali Linux (virtual machine) |
| Virtualization | VirtualBox or VMware |
| Isolation | VM with no access to host disk |

> **Why a virtual machine?**
> The `dd` command operates directly on block devices with no confirmation prompt. Swapping `if` and `of` parameters overwrites the entire host disk with no recovery possible. Running inside a VM eliminates this risk — the worst case is restoring a snapshot.

### Tools

| Tool | Function | Phase |
|---|---|---|
| `dd` | Raw forensic image creation (.dd) | Acquisition |
| `ewfacquire` | Image creation in .E01 format | Acquisition |
| `md5sum` / `sha256sum` | Integrity verification (hash) | Chain of custody |
| `mmls` (TSK) | Partition identification | Analysis |
| `fls` (TSK) | File listing including deleted files | Analysis |
| `istat` (TSK) | Inode metadata analysis | Analysis |
| `icat` (TSK) | File recovery by inode number | Analysis |
| `blkls` (TSK) | Unallocated space extraction | Analysis |
| `mactime` (TSK) | Forensic timeline generation | Timeline |
| `foremost` | Signature-based file carving | Recovery |
| `exiftool` | EXIF metadata analysis | Metadata |
| `strings` + `grep` | String search in binary image | Search |
| Autopsy | Integrated visual analysis | Consolidation |

---

## 4. Project Architecture

```
SYNTHETIC EVIDENCE
      │
      ▼
┌─────────────────────────────────────────┐
│         ACQUISITION PHASE               │
│  dd → evidencia.dd                      │
│  ewfacquire → evidencia.E01             │
│  md5sum / sha256sum → hashes            │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│     TECHNICAL ANALYSIS (TSK + CLI)      │
│  mmls → partitions and offsets          │
│  fls  → file structure + deleted files  │
│  istat → inode metadata                 │
│  icat  → file recovery                  │
│  blkls → unallocated space              │
│  strings + grep → pattern search        │
└─────────────────┬───────────────────────┘
                  │
          ┌───────┴───────┐
          ▼               ▼
┌──────────────┐   ┌──────────────────┐
│   CARVING    │   │  EXIF METADATA   │
│  foremost    │   │  exiftool        │
└──────┬───────┘   └────────┬─────────┘
       │                    │
       └─────────┬──────────┘
                 ▼
┌─────────────────────────────────────────┐
│         FORENSIC TIMELINE               │
│  fls -m → bodyfile                      │
│  mactime → chronological timeline       │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│     VISUAL ANALYSIS — AUTOPSY           │
│  Image import                           │
│  Ingest Modules                         │
│  Results / Extracted Content            │
│  Deleted Files / Timeline               │
└─────────────────┬───────────────────────┘
                  │
                  ▼
┌─────────────────────────────────────────┐
│         FORENSIC REPORT                 │
│  Methodology / Evidence / Findings      │
│  Technical conclusion                   │
└─────────────────────────────────────────┘
```

---

## 5. Project Phases

> Each phase has detailed documentation in the [`docs/`](docs/) folder.

---

### Phase 1 — Environment Setup

Installation and verification of all tools on Kali Linux.

```bash
sudo apt update
sudo apt install sleuthkit ewf-tools autopsy exiftool foremost -y

# Verify installation
fls -V
mmls -V
exiftool -ver
foremost -V
```

📄 Full documentation: [docs/phase1-environment-setup.md](docs/phase1-environment-setup.md)

---

### Phase 2 — Synthetic Evidence Creation

A 200MB raw disk image was created using `dd`, partitioned, formatted as ext4, and mounted via loopback. Files simulating a fraud scenario were written to the partition, then intentionally deleted to replicate evidence destruction.

```bash
# Create 200MB blank image
dd if=/dev/zero of=evidencias/evidencia.dd bs=1M count=200 status=progress

# Partition and format
parted evidencias/evidencia.dd mklabel msdos
parted evidencias/evidencia.dd mkpart primary ext4 1MiB 199MiB
sudo losetup -Pf evidencias/evidencia.dd
sudo mkfs.ext4 /dev/loop0p1
sudo mount /dev/loop0p1 /mnt/forense

# Plant simulated evidence files
echo "PIX Transfer R$ 5000 CPF 123.456.789-00" > /mnt/forense/transfer_receipt.txt
echo "access_password: admin@2024" > /mnt/forense/credentials.txt
cp /usr/share/pixmaps/kali-menu.png /mnt/forense/meeting_photo.png

# Simulate evidence destruction
rm /mnt/forense/transfer_receipt.txt
rm /mnt/forense/credentials.txt

# Unmount
sudo umount /mnt/forense
sudo losetup -d /dev/loop0
```

> File deletion in Linux removes only the directory entry. The actual data remains on disk until overwritten — the foundation of forensic file recovery demonstrated in Phases 4 and 5.

📄 Full documentation: [docs/phase2-evidence-creation.md](docs/phase2-evidence-creation.md)

---

### Phase 3 — Forensic Acquisition

Acquisition is the most critical phase. Any modification to the original evidence after this point may compromise its legal validity.

```bash
# Generate hashes for chain of custody
md5sum evidencias/evidencia.dd > evidencias/evidencia.md5
sha256sum evidencias/evidencia.dd >> evidencias/evidencia.md5

# Convert to E01 format
ewfacquire evidencias/evidencia.dd -t evidencias/evidencia_e01
```

> The `.dd` format is universal and compatible with all tools. The `.E01` (EnCase Evidence File) adds case metadata, compression, and internal integrity verification — standard in legally accepted forensic reports.

📄 Full documentation: [docs/phase3-forensic-acquisition.md](docs/phase3-forensic-acquisition.md)

---

### Phase 4 — Analysis with The Sleuth Kit (TSK)

TSK is the forensic engine that Autopsy uses internally. Running it directly from the command line reveals what happens beneath the graphical interface.

```bash
# Identify partitions and offsets
mmls evidencias/evidencia.dd

# List all files including deleted (marked with *)
fls -o 2048 evidencias/evidencia.dd

# List only deleted files
fls -d -o 2048 evidencias/evidencia.dd

# Recover deleted file by inode number
icat -o 2048 evidencias/evidencia.dd 13 > recuperados/transfer_receipt_recovered.txt

# Search for patterns in the binary image
strings evidencias/evidencia.dd | grep -i "PIX"

# Extract unallocated space
blkls -o 2048 evidencias/evidencia.dd > analise/unallocated.raw
```

📄 Full documentation: [docs/phase4-tsk-analysis.md](docs/phase4-tsk-analysis.md)

---

### Phase 5 — File Carving and Recovery

Carving recovers files directly from binary disk data, without relying on the file system table — useful when inodes have been overwritten or corrupted.

```bash
# Carving by file signature (magic number)
foremost -i evidencias/evidencia.dd -o recuperados/foremost -v

# Check what was recovered
cat recuperados/foremost/audit.txt
```

> Every file format has specific bytes at the start that identify its true type regardless of extension. JPEG files always begin with `FF D8 FF`. Foremost uses these signatures to locate and reconstruct files even without a file system structure.

📄 Full documentation: [docs/phase5-file-carving.md](docs/phase5-file-carving.md)

---

### Phase 6 — Metadata and EXIF Analysis

Metadata is data about data. In image files, the EXIF standard can reveal when, where, and with which device a photo was taken — information suspects frequently overlook when attempting to hide evidence.

```bash
# Full EXIF metadata analysis
exiftool recuperados/foremost/png/meeting_photo.png > metadados/exiftool_output.txt
cat metadados/exiftool_output.txt
```

📄 Full documentation: [docs/phase6-exif-metadata.md](docs/phase6-exif-metadata.md)

---

### Phase 7 — Forensic Timeline

The forensic timeline is the chronological reconstruction of all events recorded in file system metadata — the primary tool for answering "what happened and when".

```bash
# Generate bodyfile with all timestamps (MACB)
fls -m / -r -o 2048 evidencias/evidencia.dd > timeline/bodyfile.txt

# Generate chronological timeline
mactime -b timeline/bodyfile.txt > timeline/timeline.txt
```

> MACB timestamps — Modified, Accessed, Changed, Born — record every interaction with a file at the filesystem level, independently of the file's content.

📄 Full documentation: [docs/phase7-forensic-timeline.md](docs/phase7-forensic-timeline.md)

---

### Phase 8 — Visual Analysis with Autopsy

Autopsy consolidates graphically everything done manually in the previous phases, adding automatic analysis modules (ingest modules) and artifact correlation.

**Procedure:**
1. Open Autopsy → `New Case`
2. Fill in: Case Name, Base Directory, Examiner
3. `Add Data Source` → `Disk Image or VM File` → select `evidencia.dd`
4. Enable Ingest Modules: Hash Lookup, Recent Activity, Keyword Search, File Type Identification, Embedded File Extractor, EXIF Parser
5. Wait for full processing
6. Explore: Data Sources, Deleted Files, Extracted Content, Keyword Hits, Timeline

> Everything Autopsy presents visually corresponds to what TSK executes internally — `mmls`, `fls`, `icat`, `istat`. Understanding TSK means understanding what Autopsy does under the hood.

📄 Full documentation: [docs/phase8-autopsy.md](docs/phase8-autopsy.md)

---

### Phase 9 — Forensic Report Production

The report is the final product of the investigation. It must be technically precise, legally grounded, and understandable by non-technical readers.

📄 Full documentation: [docs/phase9-forensic-report.md](docs/phase9-forensic-report.md)

---

## 6. Investigation Scenario

**Hypothesis:** suspected financial fraud via PIX transfer. The investigated device belongs to the suspect and may contain transfer receipts, credentials, and communication records related to the scheme.

**Forensic questions:**

1. Are there deleted files that could evidence the fraud?
2. Is it possible to recover transfer receipts?
3. Do file metadata confirm the incident's time window?
4. Are there signs of evidence destruction attempts?

---

## 7. Repository Structure

```
digital-forensics-investigation/
│
├── README.md                        ← this document
│
├── docs/
│   ├── phase1-environment-setup.md
│   ├── phase2-evidence-creation.md
│   ├── phase3-forensic-acquisition.md
│   ├── phase4-tsk-analysis.md
│   ├── phase5-file-carving.md
│   ├── phase6-exif-metadata.md
│   ├── phase7-forensic-timeline.md
│   ├── phase8-autopsy.md
│   └── phase9-forensic-report.md
│
├── evidencias/
│   ├── evidencia.dd                 ← raw forensic image
│   ├── evidencia_e01.E01            ← EnCase format image
│   └── evidencia.md5                ← integrity hashes
│
├── analise/
│   ├── mmls_output.txt              ← identified partitions
│   ├── fls_output.txt               ← file listing
│   ├── fls_deleted.txt              ← deleted files
│   ├── strings_grep.txt             ← pattern search results
│   └── unallocated.raw              ← extracted unallocated space
│
├── recuperados/
│   ├── transfer_receipt_recovered.txt
│   └── foremost/                    ← carving results
│
├── metadados/
│   └── exiftool_output.txt          ← extracted EXIF metadata
│
├── timeline/
│   ├── bodyfile.txt
│   └── timeline.txt                 ← chronological timeline
│
├── screenshots/
│   ├── phase1_apt_install.png
│   ├── phase1_tool_versions.png
│   ├── phase2_dd_creation.png
│   ├── phase2_parted.png
│   ├── phase2_files_created.png
│   ├── phase2_files_deleted.png
│   └── phase2_unmount.png
│
└── laudo/
    └── forensic_report.pdf          ← final report
```

---

## 8. Technical and Regulatory References

- **ISO/IEC 27037:2012** — Identification, collection, acquisition and preservation of digital evidence
- **ISO/IEC 27041, 27042, 27043** — Analysis and interpretation of digital evidence
- **RFC 3227** — Guidelines for Evidence Collection and Archiving
- **NIST SP 800-86** — Guide to Integrating Forensic Techniques into Incident Response
- **Brazilian Criminal Procedure Code — Art. 158-A to 158-F** — Chain of custody of evidence
- **Brian Carrier — File System Forensic Analysis** — technical reference for TSK and Autopsy
- **The Sleuth Kit Documentation** — https://www.sleuthkit.org
- **Autopsy Documentation** — https://www.autopsy.com/documentation

---

## 9. About the Author

Project developed as part of a Digital Forensics and Expert Examination mentorship program, with the goal of consolidating technical knowledge and building a public portfolio demonstrating practical competence in the main tools and methodologies of the field.

---

*This document was produced with technical and academic rigor. All procedures were performed in a controlled virtualized environment, with no access to real third-party devices.*
