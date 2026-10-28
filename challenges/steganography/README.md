
# Steganography — DEADFACE CTF 2026

## Overview

This collection documents steganography challenges from DEADFACE CTF 2026, including image analysis, hidden information, metadata inspection, and digital forensic techniques.

Each investigation includes the challenge methodology, supporting evidence, technical explanations, security implications, and lessons learned.

## Challenges

| Challenge | Points | Status | Write-up |
|---|---:|---|---|
| Headshot | 10 | ✅ Team Solve — Independently Reproduced | [View Write-up](headshot/README.md) |
| File Dump 01 | 10 | ✅ Solved — Personal Solve | [View Write-up](file-dump-01/README.md) |
| Hidden Loot Box | 15 | 📝 Reconstructed — Resource Offline | [View Write-up](hidden-loot-box/README.md) |
| Exfiltration | 75 | ✅ Flag Recovered — Post-CTF Investigation | [View Write-up](exfiltration/README.md) |
| Conduct Unbecoming | 75 | ✅ Solved Post-CTF | [View Write-up](conduct-unbecoming/README.md) |

## Techniques Explored

### Image Metadata Analysis

[Headshot](headshot/README.md) — Identified a hidden flag within a PNG Comment metadata field using ExifTool.

### File Integrity & Hash Comparison

[Headshot](headshot/README.md) — Compared SHA-256 hashes of original and modified image files to establish that their contents differed.

### Digital Forensics on macOS

[Headshot](headshot/README.md) — Independently reproduced a teammate's confirmed challenge solution using ExifTool 13.55 rather than FTK Imager.

## Tools & Technologies

| Tool | Purpose |
|---|---|
| ExifTool 13.55 | Inspect and extract PNG metadata |
| `file` | Identify image formats and properties |
| `shasum` | Calculate SHA-256 hashes |
| macOS Terminal | Evidence examination and command execution |
| Git & GitHub | Version control and investigation documentation |

### Document Metadata Analysis & Evidence Correlation

[File Dump 01](file-dump-01/README.md) — Identified a DEADFACE member's alias by analyzing creator and last-modifier metadata across Microsoft Office documents using ExifTool.

### Web Reconnaissance & robots.txt Enumeration

[Hidden Loot Box](hidden-loot-box/README.md) — Reconstructed a challenge involving `robots.txt` discovery, navigation to an unlinked webpage, and recovery of a flag displayed within an image. The original challenge server is no longer accessible.

### EXIF Metadata & Chronological Reconstruction

[Exfiltration](exfiltration/README.md) — Recovered a message distributed across 20 JPEG images by sorting EXIF `DateTimeOriginal` values and decoding hexadecimal ASCII fragments stored in `ImageUniqueID`.

### Hexadecimal Decoding & Python Automation

[Exfiltration](exfiltration/README.md) — Developed a Python script using ExifTool's JSON output to reconstruct the hidden flag automatically.

## Investigation Approach

Challenges are investigated using a structured forensic methodology:

1. Identify and preserve the provided evidence.
2. Examine file types, metadata, and structure.
3. Investigate anomalies and hidden information.
4. Verify findings using reproducible commands.
5. Document results, security implications, and lessons learned.

Competition solves and independently reproduced investigations are distinguished from unsolved challenges and post-CTF reference studies.

---

[← Return to DEADFACE CTF 2026](../../README.md)
