# Headshot — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Steganography |
| Challenge | Headshot |
| Points | 10 |
| Challenge Author | @syyntax |
| Status | ✅ Solved |
| Personal Verification | ✅ Independently Reproduced |
| Primary Tool | ExifTool 13.55 |
| Operating System | macOS |
| Technique | PNG Metadata Analysis |

> **Competition Attribution:** This challenge was originally completed and confirmed by a teammate during DEADFACE CTF 2026. I subsequently reproduced the investigation on macOS using ExifTool, independently extracting and verifying the hidden flag. The original competition solve is credited to Jed.

---

## 1. Challenge Description & Objective

### Scenario

DEADFACE was able to compromise a fileshare that consisted of corporate headshots for Aurora Health employees. We believe they're tagging the photos with information that communicates to their group which employees to target in their future campaigns. Figure out which image DEADFACE tampered with - it will give us a heads up on who they plan to target.

### Objective

Examine the provided images for evidence of tampering and identify the hidden flag.

**Required flag format:**

`deadface{flag_text}`

### Evidence Provided

The extracted challenge files included:

```text
headshot/
├── headshot001.png
├── headshot002.png
├── headshot003.png
└── headshot003.png_original
```

The presence of both `headshot003.png` and `headshot003.png_original` made Headshot 003 a particularly useful comparison candidate.

The investigation focused on detecting differences between the suspected modified image and its original counterpart.

---

## 2. Technical Knowledge & Concepts

### 2.1 Steganography and Metadata-Based Hiding

Steganography involves concealing information within another file, message, or medium.

In image-related challenges, hidden data might be stored in:

- Image pixels
- Color channels
- Least significant bits
- File headers or metadata
- Embedded or appended file content

This challenge used a simpler approach: storing the flag inside a PNG metadata comment.

The flag was not recovered through pixel analysis or visual inspection.

### 2.2 PNG File Structure

Portable Network Graphics (PNG) files contain structured chunks that store image data and supporting information.

PNG files can also contain textual metadata.

A `Comment` field can store readable text without changing the image's displayed dimensions.

### 2.3 ExifTool

ExifTool is a command-line utility used to inspect and manage metadata in files.

It supports many formats, including PNG, JPEG, TIFF, PDF, and numerous other media types.

The following options were relevant:

| Option | Purpose |
|---|---|
| `-a` | Display duplicate metadata tags |
| `-G1` | Show metadata group names |
| `-s` | Display short tag names |
| `-Comment` | Extract the Comment metadata field |

### 2.4 Cryptographic Hashing

SHA-256 produces a 256-bit digest that can be used to compare file integrity.

Two files with different SHA-256 hashes are not byte-for-byte identical.

Hash differences alone do not reveal what was modified; additional analysis is required.

---

## 3. Investigation Methodology

### Prepare the Evidence Workspace

A dedicated workspace was used to separate investigation files from the GitHub repository.

```bash
mkdir -p ~/CTF-Evidence/Steganography/Headshot
cd ~/CTF-Evidence/Steganography/Headshot
```

The supplied files were available in the `headshot/` subdirectory.

### Enumerate the Files

I listed the extracted evidence:

```bash
ls -lah headshot/
```

The directory contained three numbered headshots and a separate file marked as the original version of Headshot 003.

I then identified the file types:

```bash
find headshot/ -type f -exec file {} \;
```

**Results:**

```text
headshot/headshot002.png: PNG image data, 512 x 512, 8-bit colormap, non-interlaced
headshot/headshot003.png: PNG image data, 512 x 512, 8-bit colormap, non-interlaced
headshot/headshot001.png: PNG image data, 512 x 512, 8-bit colormap, non-interlaced
headshot/headshot003.png_original: PNG image data, 512 x 512, 8-bit colormap, non-interlaced
```

All four files were valid, palette-based PNG images with identical dimensions.

This established a consistent baseline for further examination.

### Compare SHA-256 Hashes

Because the archive contained two versions of Headshot 003, I calculated their SHA-256 hashes:

```bash
shasum -a 256 headshot/headshot003.png headshot/headshot003.png_original
```

**Results:**

| File | SHA-256 |
|---|---|
| `headshot003.png` | `4e72015a735f4d92184ee61030539b8a298e1fb9822a5f656b35155a3a645a3e` |
| `headshot003.png_original` | `9eaf5191f4aaf5e8666e5950c8d11e26a6a63367127867234b5fb7fff540b9ef` |

The hashes differed.

This proved that the files were not byte-for-byte identical, even though they shared the same dimensions and image format.

The next step was to identify the difference responsible for the suspicious content.

### Inspect Image Metadata

I used ExifTool to display the metadata of both images:

```bash
exiftool -a -G1 -s headshot/headshot003.png headshot/headshot003.png_original
```

The modified image contained a significant field:

```text
[PNG] Comment : deadface{b00m_h34Dsh0TTT}
```

The corresponding original image did not contain a `Comment` field.

Both files otherwise reported common structural properties:

| Property | Modified | Original |
|---|---|---|
| Format | PNG | PNG |
| Width | 512 | 512 |
| Height | 512 | 512 |
| Bit depth | 8 | 8 |
| Color type | Palette | Palette |
| Interlace | Noninterlaced | Noninterlaced |
| Comment field | Present | Absent |

This was the decisive finding.

### Verify the Hidden Flag

I extracted only the comment from the modified image:

```bash
exiftool -Comment headshot/headshot003.png
```

**Verified output:**

```text
Comment : deadface{b00m_h34Dsh0TTT}
```

I then checked the original image:

```bash
exiftool -Comment headshot/headshot003.png_original
```

**Result:** No output.

The lack of a `Comment` field in the original provided further evidence that the suspicious metadata belonged to the modified image.

The flag was recovered directly from the file without needing to alter it.

---

## 5. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Flag:**

`deadface{b00m_h34Dsh0TTT}`

**Competition Status:** ✅ Solved and confirmed by the team.

**Independent Reproduction:** ✅ Flag recovered using ExifTool on macOS.

</details>

---

[← Return to Steganography Collection](../README.md)
