# Phase 6 — Metadata and EXIF Analysis

## Context

Metadata is data about data. Every digital file carries embedded metadata that
records information about its origin, creation, modification history, and in the
case of images — the device, settings, and geographic location where it was produced.

Suspects frequently focus on deleting file content while overlooking the metadata
embedded in the files they keep or fail to fully erase. This makes metadata analysis
one of the most productive phases of a digital investigation.

## What Was Done

ExifTool was used to extract all available metadata from the PNG file recovered
in Phase 5 (file carving). Specific forensically relevant fields were queried
individually — GPS coordinates, device information, and modification timestamps —
and the full output was saved to `metadata/exiftool_output.txt`.

## Commands Executed

```bash
# Full metadata extraction — saved to file
exiftool recovered/foremost/png/00020486.png > metadata/exiftool_output.txt

# Extract specific forensically relevant fields
exiftool -CreateDate -FileModifyDate -Make -Model recovered/foremost/png/00020486.png

# Check for GPS coordinates
exiftool -GPSLatitude -GPSLongitude -GPSAltitude recovered/foremost/png/00020486.png

# Detect signs of metadata tampering
exiftool -all recovered/foremost/png/00020486.png | grep -i "modify"
```

## Findings

### Specific fields (`-CreateDate -FileModifyDate -Make -Model`)
```
File Modification Date/Time : 2026:06:09 17:41:12-04:00
```
Only `FileModifyDate` returned a value — the timestamp when Foremost wrote the
recovered file to disk. `CreateDate`, `Make`, and `Model` were absent because
this file was not captured by a camera or smartphone.

### GPS coordinates
```
(no output)
```
No GPS data present in this file.

### Tampering check (`grep -i "modify"`)
```
Modify Date  : 2026:06:07 16:29:38
Datemodify   : 2026-06-07T16:29:38+00:00
```
Both modification fields point to **June 7, 2026** — consistent with the file
creation date in Phase 2. No temporal anomalies detected. The difference between
the internal `Modify Date` (June 7) and the `File Modification Date/Time` (June 9)
is expected: the file was created on June 7 and written to disk by Foremost on June 9
during carving. This chronological consistency is itself a finding — it indicates
the metadata has not been tampered with.

---

## Important Note — Why GPS and Device Fields Returned No Data

> The image analyzed in this phase (`00020486.png`) was a system-generated PNG
> file used as a placeholder during the lab setup. It was not captured by a camera
> or smartphone, which is why fields like GPS coordinates, device make/model, and
> camera serial number returned no values.
>
> **This was intentional and part of the learning process** — running these commands
> against a file with no EXIF data demonstrates exactly what absence of evidence
> looks like, which is equally important to recognize in a real investigation.
>
> **In a real forensic investigation**, the same commands against an image captured
> by a smartphone would typically return:
>
> | Field | Example value | Forensic significance |
> |---|---|---|
> | GPS Latitude / Longitude | `23°32'48.0"S 46°37'56.0"W` | Exact location where the photo was taken |
> | GPS Altitude | `760 m Above Sea Level` | Elevation — useful for corroboration |
> | Make / Model | `Apple / iPhone 14 Pro` | Links the image to a specific device |
> | Camera Serial Number | `DNXXXXXXX` | Links the image to a specific physical unit |
> | Create Date | `2024:11:15 14:32:07` | When the shutter was pressed |
> | Software | `Adobe Lightroom 6.0` | Indicates post-processing — potential tampering |
> | Lens Info | `2.22mm f/1.78` | Additional device fingerprinting |
>
> GPS coordinates extracted from a suspect's image can be plotted directly on a map
> to place them at a specific location at a specific time — one of the most powerful
> corroborating findings in digital forensics.
>
> The absence of GPS data in this lab file does not reflect a limitation of the
> methodology — it reflects the nature of the test file. The commands, the workflow,
> and the interpretation of results are identical regardless of whether data is
> present or absent.

---

## Key Concepts

**What is EXIF?**
EXIF (Exchangeable Image File Format) is a standard that specifies metadata formats
for images captured by digital cameras and smartphones. It is embedded directly into
JPEG, TIFF, and PNG files and is invisible to the average user — which is precisely
what makes it forensically valuable.

**Metadata vs. filesystem timestamps**
There are two distinct layers of timestamps in digital forensics:

- **Filesystem timestamps** (from `istat` in Phase 4) — recorded by the operating
  system: when the file was created, accessed, or modified on that specific machine
- **Embedded metadata timestamps** (from ExifTool) — recorded inside the file itself
  by the camera or application that created it, and travel with the file regardless
  of where it is copied or moved

A suspect can modify filesystem timestamps using tools like `touch`. Embedded EXIF
timestamps are harder to alter without leaving traces — and inconsistency between
the two layers is itself a forensic finding.

**Detecting tampering**
Signs of metadata tampering include:
- `Modify Date` earlier than `Create Date` — physically impossible without manipulation
- `Software` field showing an editor on a file claimed to be an original capture
- GPS coordinates inconsistent with the claimed location of the suspect

**ExifTool scope**
ExifTool supports over 100 file formats — images, videos, PDFs, Office documents,
audio files — making it applicable far beyond image analysis in a full investigation.

## Screenshots

- `screenshots/phase6_exiftool.png` — complete metadata output
- `screenshots/phase6_specific_fields.png` — specific field queries and results
