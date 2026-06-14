# Forensic Examination Report

**Case Number:** 001  
**Report Date:** June 13, 2026  
**Examiner:** Paulo Vaz — Digital Forensics Analyst  
**Institution:** Aissa Tecnologia da Informação  

---

## 1. Identification

| Field | Details |
|---|---|
| Case Number | 001 |
| Case Name | Digital Forensics Lab |
| Examiner | Paulo Vaz — Digital Forensics Analyst |
| Examination Date | June 7–13, 2026 |
| Report Date | June 13, 2026 |
| Environment | Kali Linux (VirtualBox VM) + Autopsy 4.23.1 (Windows host) |

---

## 2. Object of Examination

The object of this examination is a 200MB raw disk image (`evidence.dd`) and its
EnCase-format counterpart (`evidence_e01.E01`), created in a controlled laboratory
environment to simulate a suspect device involved in a financial fraud scenario.

The examination was commissioned to answer the following forensic questions:

1. Are there deleted files on the device that may constitute evidence of fraud?
2. Is it possible to recover the content of deleted files?
3. Do file system timestamps confirm the incident's time window?
4. Are there signs of deliberate evidence destruction?

---

## 3. Methodology

This examination was conducted in accordance with internationally recognized digital
forensics standards and best practices:

- **ISO/IEC 27037:2012** — Identification, collection, acquisition and preservation of digital evidence
- **ISO/IEC 27041 / 27042 / 27043** — Analysis and interpretation of digital evidence
- **RFC 3227** — Guidelines for Evidence Collection and Archiving
- **NIST SP 800-86** — Guide to Integrating Forensic Techniques into Incident Response

### Tools Used

| Tool | Version | Purpose |
|---|---|---|
| `dd` | GNU coreutils | Raw disk image creation |
| `ewfacquire` / `ewfverify` | libewf 20140816 | E01 acquisition and verification |
| `md5sum` / `sha256sum` | GNU coreutils | Integrity hashing |
| The Sleuth Kit (`mmls`, `fls`, `istat`, `icat`, `blkls`, `mactime`) | 4.14.0 | File system analysis |
| `foremost` | 1.5.7 | Signature-based file carving |
| `exiftool` | 13.55 | Metadata and EXIF extraction |
| `strings` + `grep` | GNU binutils | Raw string search |
| Autopsy | 4.23.1 | Integrated graphical analysis |

### Chain of Custody

Cryptographic hashes were calculated immediately after image creation and verified
prior to analysis. Any alteration to the image would produce a different hash value,
invalidating the evidence.

| Algorithm | Hash Value |
|---|---|
| MD5 | `5b98c6bdd1c195a11c8e2d6958dab25c` |
| SHA256 | `d8d0d4b942f4dfd433189ae64aa4058599b5587a3018b210eb6c5bcb11d6edee` |

Hashes were verified before and after analysis:
- `md5sum -c evidences/evidence.md5` → **OK**
- `sha256sum -c evidences/evidence.md5` → **OK**

---

## 4. Evidence Examined

| Item | Description | Format | Size | Integrity |
|---|---|---|---|---|
| `evidence.dd` | Raw forensic disk image | RAW (.dd) | 200 MB | Verified (MD5 + SHA256) |
| `evidence_e01.E01` | EnCase format image | E01 | 201 MB | Verified (ewfverify) |

---

## 5. Technical Analysis

### 5.1 Partition Analysis

Partition layout identified using `mmls`:

```
002:  000:000   0000002048   0000409599   Linux (0x83)
```

A single Linux (ext4) partition was identified, starting at sector **2048**.
All subsequent analysis used this offset.

### 5.2 File System Analysis

Active files identified via `fls`:

| Inode | Name | Status | Size |
|---|---|---|---|
| 11 | lost+found | Allocated | 12288 bytes |
| 15 | foto_reuniao.png | Allocated | 442 bytes |
| 16 | log_sistema.log | Allocated | 233 bytes |

Deleted files identified via `fls -r`:

| Inode | Name | Status |
|---|---|---|
| 13 | OrphanFile-13 (transfer_receipt.txt) | Deleted |
| 14 | OrphanFile-14 (credentials.txt) | Deleted |

### 5.3 Inode Metadata Analysis

Metadata inspection via `istat` for deleted inodes:

**Inode 13 (transfer_receipt.txt)**
- Created: 2026-06-07 12:26:08
- Deleted: 2026-06-07 12:33:09
- Size: 0 (blocks zeroed by vboxsf at deletion)

**Inode 14 (credentials.txt)**
- Created: 2026-06-07 12:27:03
- Deleted: 2026-06-07 12:33:09
- Size: 0 (blocks zeroed by vboxsf at deletion)

> Although file content was not recoverable via inode (blocks zeroed by the
> VirtualBox shared folder filesystem), the timestamps constitute valid forensic
> evidence that both files existed and were deliberately deleted at 12:33:09.

### 5.4 Raw String Search

Direct binary search via `strings + grep` recovered the following data from the
raw image, independent of file system structure:

```
Transferencia PIX R$ 5000 CPF 123.456.789-00
2026-06-07 12:24:03 TRANSFERENCIA PIX valor=5000
```

Search of 181MB of unallocated space (extracted via `blkls`) recovered:

```
senha_acesso: admin@2024
```

These findings confirm that file deletion did not erase the underlying data —
content remained physically present on the disk and was recoverable through
raw binary analysis.

### 5.5 File Carving

Foremost performed signature-based carving across the entire 200MB image:

```
0:  00020486.png   442 B   offset 10488832   (640 x 480)
1 FILES EXTRACTED
```

One PNG file was recovered at byte offset 10,488,832. The file is intact and
valid (640×480 pixels, 442 bytes), corresponding to the image file present
on the device.

### 5.6 Metadata Analysis

ExifTool analysis of the recovered PNG file:

| Field | Value | Significance |
|---|---|---|
| Modify Date | 2026-06-07 16:29:38 UTC | Consistent with Phase 2 creation |
| Datemodify | 2026-06-07T16:29:38+00:00 | Corroborates Modify Date |
| File Modification Date | 2026-06-09 17:41:12 | Written by Foremost during carving |
| GPS Coordinates | Not present | File was system-generated |
| Make / Model | Not present | File was system-generated |

No temporal anomalies detected. The difference between internal modification date
(June 7) and file system modification date (June 9) is consistent with the file
having been created on June 7 and recovered by Foremost on June 9.

### 5.7 Forensic Timeline

Chronological reconstruction via `fls -m` and `mactime`:

| Time (EDT) | Event | Significance |
|---|---|---|
| 12:12:15 | lost+found created | Filesystem initialized |
| 12:26:08 | OrphanFile-13 created | transfer_receipt.txt written to disk |
| 12:27:03 | OrphanFile-14 created | credentials.txt written to disk |
| 12:29:38 | foto_reuniao.png — full MACB | Image file created |
| 12:32:00 | log_sistema.log — full MACB | System log generated |
| 12:33:09 | OrphanFile-13 and 14 — m.c. | **Both files deleted simultaneously** |

The timeline establishes a clear sequence: two sensitive files were created between
12:26 and 12:27, and deleted 7 minutes later at 12:33 — consistent with deliberate
evidence destruction.

### 5.8 Autopsy Consolidated Analysis

All findings were confirmed through independent analysis in Autopsy 4.23.1:

- Deleted Files view identified OrphanFile-13 and OrphanFile-14 with matching timestamps
- File Metadata panel confirmed inode 15 (`foto_reuniao.png`) with MD5 and SHA256 hashes
- Visual timeline corroborated the chronological sequence from Phase 7
- Full HTML report generated and preserved at `laudo/autopsy_report/report.html`

---

## 6. Conclusions

Based on the technical analysis performed, the following conclusions are drawn
in response to the forensic questions posed:

**1. Are there deleted files that may constitute evidence of fraud?**
Yes. Two files — `transfer_receipt.txt` (inode 13) and `credentials.txt` (inode 14)
— were identified as deleted. Their existence and deletion are confirmed by inode
metadata, forensic timeline, and Autopsy analysis.

**2. Is it possible to recover the content of deleted files?**
Partially. Inode-based recovery via `icat` was unsuccessful due to block zeroing
by the VirtualBox shared folder filesystem — an environment-specific limitation.
However, file content was recovered through raw string search (`strings + grep`)
and unallocated space analysis (`blkls`), confirming the presence of PIX transfer
data, a CPF number, and credential information.

**3. Do file system timestamps confirm the incident's time window?**
Yes. MACB timestamps recorded in inode metadata and confirmed by the forensic
timeline establish that both sensitive files were created between 12:26 and 12:27
on June 7, 2026, and deleted at 12:33 on the same date.

**4. Are there signs of deliberate evidence destruction?**
Yes. The simultaneous deletion of both files at 12:33:09 — 7 minutes after creation
— is consistent with deliberate evidence destruction. The pattern of creating
sensitive files and deleting them shortly after is a behavioral indicator commonly
observed in fraud investigations.

---

## 7. Limitations

- Inode-based content recovery (`icat`) was unsuccessful due to block zeroing
  by the VirtualBox shared folder filesystem (vboxsf). This is an
  environment-specific behavior and does not reflect the methodology's capability
  on physical devices or native Linux filesystems.
- The evidence image was synthetically created for laboratory purposes. Findings
  are forensically valid within the scope of this controlled environment.
- No GPS or device metadata was present in the recovered image file, as it was
  system-generated rather than captured by a camera or smartphone.

---

## 8. Recommendations

- Preserve all evidence files with verified hash values
- In cases involving physical devices, use a hardware write blocker during acquisition
- For credential data recovered (`admin@2024`), initiate immediate access revocation
  procedures if applicable
- Cross-reference PIX transfer data with financial institution records via legal request

---

## 9. Examiner Declaration

I declare that this report was produced based solely on the evidence examined,
using recognized digital forensics methodologies, and that the findings described
herein are an accurate and objective representation of the technical analysis performed.

**Examiner:** Paulo Vaz — Digital Forensics Analyst  
**Institution:** Aissa Tecnologia da Informação  
**Date:** June 13, 2026  

---

*This report was produced in a controlled laboratory environment for educational
and portfolio purposes, as part of a Digital Forensics mentorship program.
All procedures followed internationally recognized forensic standards.*
