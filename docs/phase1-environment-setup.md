# Phase 1 — Environment Setup

## Context

Before any forensic investigation begins, the examiner must prepare a controlled,
reproducible environment with all required tools installed and verified. This phase
establishes the technical baseline for every subsequent step.

All procedures were performed inside a Kali Linux virtual machine, ensuring complete
isolation from the host system — a non-negotiable requirement when working with tools
like `dd` that operate directly on block devices.

## What Was Done

All open source forensic tools used throughout this project were installed via the
official Kali Linux repositories. Each tool was verified individually to confirm
correct installation and record the version used — essential for reproducibility
and for referencing in the final forensic report.

## Commands Executed

```bash
# Update package index
sudo apt update

# Install all required tools
sudo apt install sleuthkit ewf-tools autopsy exiftool foremost -y

# Verify each tool
fls -V
mmls -V
exiftool -ver
foremost -V
autopsy

# Save environment record
fls -V > analise/tool_versions.txt
mmls -V >> analise/tool_versions.txt
exiftool -ver >> analise/tool_versions.txt
foremost -V >> analise/tool_versions.txt
```

## Tool Overview

| Tool | Package | Purpose |
|---|---|---|
| `fls`, `mmls`, `istat`, `icat`, `blkls`, `mactime` | sleuthkit | Core forensic analysis (TSK) |
| `ewfacquire`, `ewfverify` | ewf-tools | E01 image acquisition and verification |
| Autopsy | autopsy | Graphical forensic analysis interface |
| `exiftool` | exiftool | EXIF and metadata extraction |
| `foremost` | foremost | Signature-based file carving |

## Key Concepts

**Why Kali Linux?**
Kali is a Debian-based distribution built specifically for penetration testing and
digital forensics. It ships with most forensic tools pre-packaged, ensuring
compatibility and reducing setup friction.

**Why a Virtual Machine?**
Running inside a VM provides a hard boundary between forensic operations and the
host system. If something goes wrong — wrong device targeted, accidental overwrite —
the host disk is never at risk. Snapshots also allow the environment to be restored
to any previous state instantly.

**Why record tool versions?**
Forensic reports must be reproducible. Documenting exact tool versions allows another
examiner to replicate the analysis under identical conditions, which is a requirement
under ISO/IEC 27041.

## Screenshots

- `screenshots/phase1_installed_tools.png` — apt install output confirming all packages
- `screenshots/phase1_tools_versions.png` — version output for each tool