
# Keep Digging — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Steganography |
| Challenge | Keep Digging |
| Author | @syyntax |
| Points Available | 100 |
| Competition Status | Not Solved During CTF |
| Investigation Status | Solved Post-CTF |
| Solution Classification | Externally Assisted, Independently Reproduced |
| Primary Tools | ExifTool 13.55, Python 3, Pillow, macOS Terminal |
| Techniques | JPEG Structure Analysis, File Header Manipulation, Digital Forensics |

> **Investigation Note:** Keep Digging was not solved during DEADFACE CTF 2026. I investigated the challenge after the competition, initially testing several forensic techniques without recovering the flag. I subsequently consulted a published write-up by TheMusk3teers, learned the JPEG height-manipulation technique, and independently reproduced the solution using the original challenge image.
>
> **External Source:** [TheMusk3teers — Keep Digging Write-up](https://github.com/TheMusk3teers/TheMusk3teers.github.io/blob/68b72eeed6a38d019cc18dee32be736ace8959ec/docs/ctfs/deadface-2026/steganography/keep-digging.md)

---

## 1. Challenge Description & Objective

### Scenario

DEADFACE left this calling card on a victim's machine. We suspect there is hidden information in this image, but none of our common steganography tools seem to work. There must be some way to determine where the flag is hidden.

### Objective

Analyze the supplied JPEG image, identify the concealment technique, and extract the hidden flag.

**Required flag format:**

`deadface{flag-text}`

### Evidence Provided

The challenge supplied:

```text
keep_digging.jpg
```

The original evidence was verified using its SHA-256 digest:

```text
12582c19c665ee4c7c8d4517ccb8f3559356620553970b235d8aad4e0b4ff8e2
```

### Investigation Workspace

```text
~/CTF-Evidence/Steganography/Keep-Digging/
```

Evidence and investigation artifacts included:

```text
Keep-Digging/
├── keep_digging.jpg
├── keep_digging.jpeg
├── keep_digging_fixed.jpg
├── keep_digging_red.png
├── keep_digging_green.png
├── keep_digging_blue.png
├── keep_digging_bitplanes.png
└── jpeg_structure.txt
```

The `.jpeg` file was an alternate image version encountered during analysis and did not match the challenge-provided hash.

The original, verified `keep_digging.jpg` was used for the final reconstruction.

---

## 2. Technical Concepts

### 2.1 Steganography

Steganography is the practice of concealing information inside another medium.

Digital images can contain hidden information through:

- Metadata fields
- Embedded or appended files
- Pixel-value manipulation
- Least significant bit encoding
- JPEG compression coefficients
- File header manipulation

Unlike conventional pixel-based steganography, this challenge concealed content by manipulating the JPEG's declared dimensions.

### 2.2 JPEG File Structure

JPEG files contain a sequence of structural markers describing how the image should be decoded.

Important markers include:

| Marker | Name | Purpose |
|---|---|---|
| `FF D8` | Start of Image (SOI) | Identifies the beginning of a JPEG |
| `FF EE` | APP14 | Adobe application metadata |
| `FF DB` | Define Quantization Table (DQT) | Stores quantization information |
| `FF C2` | Start of Frame 2 (SOF2) | Defines a progressive JPEG frame |
| `FF C4` | Define Huffman Table (DHT) | Stores Huffman coding tables |
| `FF DA` | Start of Scan (SOS) | Begins entropy-coded image scan data |
| `FF D9` | End of Image (EOI) | Identifies the end of the JPEG |

The SOF2 marker is particularly important because it stores the image's declared dimensions.

### 2.3 Progressive JPEG Encoding

Progressive JPEG images encode image information through multiple scans.

Rather than storing the entire image in one sequential pass, progressive JPEGs separate image information into stages.

The original challenge file used progressive DCT encoding and contained seven Start of Scan signatures.

Multiple SOS markers are normal for progressive JPEG files and do not independently establish steganography.

### 2.4 JPEG Dimension Manipulation

JPEG dimensions are stored in the Start of Frame segment.

For an SOF2 segment, the relevant layout is:

```text
FF C2 | Length (2 bytes) | Precision (1 byte)
      | Height (2 bytes) | Width (2 bytes)
      | Component information
```

If the declared height is smaller than the image data actually encoded, an image decoder may display only the declared portion.

Changing the height field can sometimes expose previously hidden image content.

This technique is commonly described as JPEG height manipulation or dimension-based steganography.

### 2.5 Hexadecimal Representation

JPEG header values are stored as binary data and are commonly inspected in hexadecimal.

For this challenge:

| Value | Hexadecimal | Decimal |
|---|---|---|
| Original image height | `0x029E` | 670 |
| Corrected image height | `0x02EE` | 750 |
| Image width | `0x01C0` | 448 |

The difference between the declared and corrected heights was:

```text
750 - 670 = 80 pixels
```

Those additional pixels contained the concealed flag.

---

## 3. Investigation Methodology

### Acquire the Original Evidence

I created a dedicated investigation workspace:

```bash
mkdir -p ~/CTF-Evidence/Steganography/Keep-Digging

cd ~/CTF-Evidence/Steganography/Keep-Digging
```

I downloaded the challenge image and identified its file format:

```bash
ls -lah
file *
```

The original evidence was identified as:

```text
keep_digging.jpg: JPEG image data, progressive,
precision 8, 448x670, components 3
```

### Evidence Characteristics

| Property | Value |
|---|---|
| Filename | `keep_digging.jpg` |
| Format | JPEG |
| Encoding | Progressive DCT |
| Dimensions | 448 × 670 pixels |
| File Size | 269,217 bytes |
| Color Components | 3 |
| Color Subsampling | YCbCr 4:4:4 |

### Inspect Image Metadata

I began with ExifTool:

```bash
exiftool -a -u -G1 -s "keep_digging.jpg"
```

Relevant findings included:

```text
FileType             : JPEG
ImageWidth           : 448
ImageHeight          : 670
EncodingProcess      : Progressive DCT, Huffman coding
YCbCrSubSampling     : YCbCr4:4:4 (1 1)
DCTEncodeVersion     : 100
ColorTransform       : YCbCr
```

The metadata contained ordinary Adobe APP14 information.

No obvious flag, comment, or concealed identifier was identified.

### Check for Appended Data

I checked whether data had been appended after the JPEG End of Image marker.

```python
from pathlib import Path

data = Path("keep_digging.jpg").read_bytes()
end = data.rfind(b"\xff\xd9")

print(f"File size: {len(data):,} bytes")
print(f"Last JPEG end marker: {end}")
print(f"Bytes after marker: {len(data) - end - 2:,}")
```

Output:

```text
File size: 269,217 bytes
Last JPEG end marker: 269215
Bytes after marker: 0
```

The file contained no trailing data after the final JPEG marker.

This ruled out a straightforward appended-payload technique.

### Examine JPEG Segments

I used ExifTool's verbose output:

```bash
exiftool -v3 "keep_digging.jpg" > jpeg_structure.txt
```

Then searched for structural markers:

```bash
grep -inE \
  'JPEG APP|JPEG COM|JPEG DQT|JPEG DHT|JPEG SOS|JPEG EOI|unknown|trailer' \
  jpeg_structure.txt | head -100
```

The output showed ordinary JPEG structures, including:

```text
JPEG APP14
JPEG DQT
JPEG DHT
JPEG SOS
JPEG EOI
```

At this stage, no structural anomaly had been identified.

### Inspect the Image Visually

I opened the original image:

```bash
open keep_digging.jpg
```

The image depicted a stylized figure wearing a gas mask and holding a shovel.

The artwork used prominent magenta, cyan, and purple tones.

No readable flag was visible.

The title **Keep Digging** suggested that additional information might be concealed beneath the displayed content, although this was not yet proven.

### Analyze RGB Color Channels

I used Python and Pillow to separate the red, green, and blue channels.

```python
from PIL import Image, ImageOps

image = Image.open("keep_digging.jpg").convert("RGB")

for name, channel in zip(
    ["red", "green", "blue"],
    image.split()
):
    enhanced = ImageOps.autocontrast(channel)
    enhanced.save(f"keep_digging_{name}.png")
```

Each channel was inspected for hidden text or structured visual patterns.

No obvious flag was identified.

### Examine Bit Planes

I also generated least significant bit-plane images.

The analysis examined bits 0 and 1 of each RGB channel.

Six bit-plane images were combined into a contact sheet.

The resulting patterns resembled JPEG compression noise and recognizable outlines from the original image.

No readable hidden message was recovered.

Because JPEG uses lossy compression, direct RGB LSB analysis does not necessarily recover data embedded in JPEG compression coefficients.

### Check for Additional JPEG Signatures

I searched for additional JPEG Start of Image signatures:

```python
from pathlib import Path

data = Path("keep_digging.jpg").read_bytes()

signature = b"\xff\xd8\xff"
offsets = []
start = 0

while True:
    pos = data.find(signature, start)

    if pos == -1:
        break

    offsets.append(pos)
    start = pos + 1

print(f"JPEG start markers found: {len(offsets)}")
print(f"Offsets: {offsets}")
```

No additional embedded JPEG start signature was identified.

### Search for Readable Strings

I examined printable strings:

```bash
strings -a -n 5 "keep_digging.jpg" | head -80
```

The results contained mostly random-looking characters associated with compressed JPEG data.

I also searched for common flag-related keywords:

```bash
strings -a -n 8 "keep_digging.jpg" \
  | grep -iE 'deadface|flag|secret|hidden|password|key'
```

No matching plaintext was returned.

### Verify Evidence Integrity

During the investigation, I encountered a second image version:

```text
keep_digging.jpeg
```

Its size and metadata differed from the initial evidence.

To avoid continuing the analysis on a potentially re-encoded image, I downloaded the original challenge file again.

I verified its SHA-256 digest:

```bash
shasum -a 256 keep_digging.jpg
```

Output:

```text
12582c19c665ee4c7c8d4517ccb8f3559356620553970b235d8aad4e0b4ff8e2  keep_digging.jpg
```

This matched the published challenge digest.

All subsequent structural reconstruction was performed using the verified original.

### Examine Progressive JPEG Scan Markers

I inspected the JPEG's Start of Scan markers.

The following offsets were identified:

```text
480
13969
26882
42864
61050
102292
163819
```

Seven scan signatures were located.

The image's progressive JPEG structure explained the presence of multiple SOS markers.

This analysis did not recover the hidden flag.

### Examine the JPEG Header

I parsed the JPEG header manually using Python.

```python
from pathlib import Path

data = Path("keep_digging.jpg").read_bytes()

print(f"File size: {len(data):,} bytes")
print(f"JPEG header: {data[:4].hex(' ')}")
```

Further inspection identified:

```text
Marker FFEE — Adobe APP14
Marker FFDB — Quantization tables
Marker FFC2 — Progressive Start of Frame
Marker FFC4 — Huffman tables
Marker FFDA — Start of Scan
```

The progressive Start of Frame marker (`FF C2`) would ultimately become the key to recovering the hidden content.

However, I had not yet established that its declared dimensions were incorrect.

### Consult an External Write-up

After several unsuccessful investigation approaches, I consulted a published solution from TheMusk3teers.

**Source:**

[TheMusk3teers — Keep Digging](https://github.com/TheMusk3teers/TheMusk3teers.github.io/blob/68b72eeed6a38d019cc18dee32be736ace8959ec/docs/ctfs/deadface-2026/steganography/keep-digging.md)

The published write-up explained that the JPEG contained more image data than its declared height permitted viewers to display.

Their analysis identified an additional 80 pixels of concealed content beneath the visible image.

This was the critical technique that I learned from the external source.

The solution method was not discovered independently during my initial analysis.

### Locate the SOF2 Marker

I independently reproduced the published technique using my verified original JPEG.

The script located the progressive JPEG Start of Frame marker:

```python
data = bytearray(open("keep_digging.jpg", "rb").read())

sof2 = data.index(b"\xff\xc2")

print("SOF2 marker offset:", sof2)
```

Output:

```text
SOF2 marker offset: 171
```

The image height is stored five bytes after the SOF2 marker's beginning.

```text
FF C2 [length:2] [precision:1] [height:2] [width:2]
```

I read the original height field:

```python
height = int.from_bytes(
    data[sof2 + 5:sof2 + 7],
    "big"
)

print("Original declared height:", height)
```

Output:

```text
Original declared height: 670
```

### Correct the JPEG Height

Following the external write-up's method, I changed the declared height from 670 pixels to 750 pixels.

I preserved the original evidence and wrote the modified bytes to a new file.

```python
from pathlib import Path

original = Path("keep_digging.jpg")
data = bytearray(original.read_bytes())

sof2 = data.index(b"\xff\xc2")

height_offset = sof2 + 5
original_height = int.from_bytes(
    data[height_offset:height_offset + 2],
    "big"
)

print(f"Original declared height: {original_height}")
print(f"SOF2 marker offset: {sof2}")

assert original_height == 670, "Unexpected image height!"

data[height_offset:height_offset + 2] = (
    750
).to_bytes(2, "big")

output = Path("keep_digging_fixed.jpg")
output.write_bytes(data)

print(f"Created: {output}")
```

Execution output:

```text
Original declared height: 670
SOF2 marker offset: 171
Created: keep_digging_fixed.jpg
```

### Recover the Hidden Image Content

I opened the corrected file:

```bash
open keep_digging_fixed.jpg
```

The corrected image displayed an additional section below the previously visible artwork.

That concealed region contained the flag:

```text
deadface{D1g_d0wN_D33p!}
```

The recovered flag matched the one reported in TheMusk3teers' published solution.

I therefore successfully reproduced the technique using my own verified challenge evidence.

---

## 4. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Recovered Flag:**

`deadface{D1g_d0wN_D33p!}`

**Post-CTF Status:** Solved and Independently Reproduced

**Competition Status:** Not Solved

**Competition Points Earned:** 0

**Solution Discovery Credit:** TheMusk3teers

</details>

---

## 5. References & Attribution

### External Solution

**TheMusk3teers — DEADFACE CTF 2026: Keep Digging**

Published solution explaining the JPEG height manipulation technique.

- [View the original write-up](https://github.com/TheMusk3teers/TheMusk3teers.github.io/blob/68b72eeed6a38d019cc18dee32be736ace8959ec/docs/ctfs/deadface-2026/steganography/keep-digging.md)
- Repository: [TheMusk3teers.github.io](https://github.com/TheMusk3teers/TheMusk3teers.github.io)
- Referenced commit: `68b72eeed6a38d019cc18dee32be736ace8959ec`

The JPEG height-manipulation method was learned from this source. My subsequent reproduction and evidence verification were performed independently.

### Technical References

- [ExifTool Documentation](https://exiftool.org/)
- [Python Documentation](https://docs.python.org/3/)
- [Pillow Documentation](https://pillow.readthedocs.io/)

---

[← Return to Steganography Collection](../README.md)
