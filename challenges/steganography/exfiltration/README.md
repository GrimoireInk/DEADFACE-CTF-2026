# Exfiltration — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Steganography |
| Challenge | Exfiltration |
| Points Available | 75 |
| Challenge Author | @G2Gh0st |
| Documentation Classification | Post-CTF Investigation |
| Primary Tools | ExifTool 13.55, Python 3 |
| Operating System | macOS |
| Techniques | EXIF Metadata Analysis, Chronological Reconstruction, Hexadecimal-to-ASCII Decoding |

## 1. Challenge Description & Objective

### Scenario

DEADFACE frequently uses unconventional methods to exchange information and avoid detection. Analysts recovered a set of images from a compromised communications channel that appear unrelated at first glance. We suspect the collection contains a hidden message intended for other members of the group. Recover the message and identify what DEADFACE is trying to say.

### Objective

Identify how information was hidden across multiple images, reconstruct the original message, and recover the flag.

**Required flag format:**

`deadface{flagtext}`

### Evidence Provided

The challenge supplied an archive containing 20 JPEG images.

After extraction, the files were organized in the following investigation workspace:

```text
~/CTF-Evidence/Steganography/Exfiltration/
├── exfiltration/
│   ├── image_01.jpg
│   ├── image_02.jpg
│   ├── ...
│   └── image_20.jpg
├── create_contact_sheet.py
├── exfiltration_contact_sheet.png
└── decode_exfiltration.py
```

The archive's original contents were preserved during the investigation.

---

## 2. Technical Knowledge & Concepts

### 2.1 Steganography

Steganography is the practice of concealing information within another medium.

In digital images, hidden information may exist in:

- Pixel values or color channels
- Least significant bits
- Image metadata
- Embedded file content
- Appended data
- Relationships between multiple files

This challenge used **metadata-based steganography distributed across multiple JPEG images**.

Instead of embedding the entire message into one image, the encoded content was divided into fragments stored in separate files.

### 2.2 EXIF Metadata

Exchangeable Image File Format (EXIF) metadata stores additional information associated with image files.

Depending on the image and its origin, EXIF fields can include camera settings, timestamps, image identifiers, software information, and other properties.

Two fields were particularly important:

| EXIF Field | Purpose in This Challenge |
|---|---|
| `DateTimeOriginal` | Determines the correct sequence of images |
| `ImageUniqueID` | Stores hexadecimal-encoded fragments of the hidden message |

Neither field necessarily appears suspicious when examining an image individually.

However, comparing the metadata across all 20 files reveals the encoding scheme.

### 2.3 Hexadecimal Encoding

Hexadecimal is a base-16 numbering system using the digits `0–9` and letters `A–F`.

Each pair of hexadecimal digits can represent one byte.

For example:

| Hexadecimal | ASCII |
|---|---|
| `64` | d |
| `65` | e |
| `61` | a |
| `7b` | { |
| `7d` | } |

Multiple byte values can be combined into a string.

For example:

```text
6465 → de
```

Hexadecimal encoding is not encryption. It is simply a way of representing data.

### 2.4 Chronological Reconstruction

In multi-file investigations, filenames do not necessarily represent the order in which content should be interpreted.

Metadata timestamps can provide an alternate ordering.

In this challenge, the filenames were shuffled relative to the timestamps, meaning that reading the hexadecimal fragments in filename order would not reconstruct the correct message.

### 2.5 Python Automation

Python was used to automate the final extraction process.

The decoder:

1. Retrieved metadata using ExifTool.
2. Parsed the metadata as JSON.
3. Sorted the images using their original timestamps.
4. Removed the `IMG_` prefix from each image identifier.
5. Converted the remaining hexadecimal values to ASCII.
6. Concatenated the decoded fragments into the complete message.

---

## 3. Investigation Methodology

### Prepare the Workspace

I created a dedicated investigation directory:

```bash
mkdir -p ~/CTF-Evidence/Steganography/Exfiltration

cd ~/CTF-Evidence/Steganography/Exfiltration
```

The challenge archive had already been extracted into the `exfiltration/` subdirectory.

### Enumerate the Evidence

I listed the extracted files:

```bash
ls -lah exfiltration/
```

The directory contained 20 numbered JPEG images:

```text
image_01.jpg
image_02.jpg
image_03.jpg
...
image_20.jpg
```

I then identified their file formats:

```bash
find exfiltration/ -type f -exec file {} \;
```

All 20 files were recognized as JPEG images.

### Evidence Characteristics

| Property | Observation |
|---|---|
| Number of files | 20 |
| Format | JPEG |
| Encoding | Baseline JPEG |
| Dimensions | Approximately 307–308 × 256 pixels |
| Approximate file size | 18–42 KB |
| Naming convention | `image_01.jpg` through `image_20.jpg` |

The consistent dimensions and filenames suggested that the images belonged to a deliberately prepared collection.

### Perform an Initial Metadata Scan

I began by searching for descriptive metadata that might contain a hidden message.

```bash
exiftool -a -G1 -s \
  -FileName -ImageDescription -Comment \
  -UserComment -Artist -Copyright \
  -Software -DateTimeOriginal \
  exfiltration/*.jpg
```

The output revealed that all 20 images contained `DateTimeOriginal` metadata.

However, the timestamps did not follow filename order.

For example:

| Filename | DateTimeOriginal |
|---|---|
| `image_01.jpg` | `2026:06:01 14:13:09` |
| `image_02.jpg` | `2026:06:01 14:35:20` |
| `image_03.jpg` | `2026:06:01 14:08:11` |
| `image_07.jpg` | `2026:06:01 14:02:17` |
| `image_14.jpg` | `2026:06:01 14:04:52` |

This suggested that the original capture timestamps might provide an alternative reading order.

At this stage, chronological ordering was only a hypothesis.

### Reconstruct the Chronological Sequence

I extracted the filenames and timestamps using ExifTool:

```bash
exiftool -csv -FileName -DateTimeOriginal exfiltration/*.jpg \
  | tail -n +2 \
  | sort -t, -k3,3
```

This arranged the images by their original timestamps.

The resulting sequence was:

```text
07 → 14 → 03 → 19 → 01
12 → 08 → 20 → 05 → 16
02 → 18 → 09 → 13 → 04
15 → 10 → 06 → 17 → 11
```

This differed significantly from the numerical filename order.

### Create a Contact Sheet

To examine the images collectively, I created a chronological contact sheet using Python and Pillow.

The script arranged the images in a 4-column by 5-row grid, labeling each image with its position and filename.

The resulting file was:

```text
exfiltration_contact_sheet.png
```

Visual inspection revealed that the images depicted nature scenes, including flowers, landscapes, forests, and waterfalls.

The images also contained visible numerical labels.

However, there was no immediately recognizable plaintext message.

The visual inspection did not establish the encoding method, so I continued investigating the metadata.

### Inspect the Complete Metadata

I examined the full metadata of the earliest image:

```bash
exiftool -a -u -G1 -s exfiltration/image_07.jpg
```

Among the returned fields were:

```text
[ExifIFD] DateTimeOriginal : 2026:06:01 14:02:17
[ExifIFD] ImageUniqueID    : IMG_6465
```

The `ImageUniqueID` field was particularly interesting.

The suffix `6465` is a valid hexadecimal string.

When interpreted as ASCII bytes:

```text
64 → d
65 → e
```

It produces:

```text
de
```

This suggested that the image identifiers might contain encoded message fragments.

### Compare ImageUniqueID Across All Files

I extracted the relevant fields from every image:

```bash
exiftool -T -FileName -DateTimeOriginal -ImageUniqueID exfiltration/*.jpg
```

The results showed that all 20 images contained `IMG_` identifiers followed by hexadecimal characters.

When the images were arranged chronologically, the identifiers formed a meaningful sequence.

### Complete Chronological Evidence Table

| Order | Image | Time | ImageUniqueID | ASCII |
|---|---|---|---|---|
| 1 | image_07.jpg | 14:02:17 | `IMG_6465` | de |
| 2 | image_14.jpg | 14:04:52 | `IMG_61` | a |
| 3 | image_03.jpg | 14:08:11 | `IMG_6466` | df |
| 4 | image_19.jpg | 14:11:36 | `IMG_61` | a |
| 5 | image_01.jpg | 14:13:09 | `IMG_6365` | ce |
| 6 | image_12.jpg | 14:18:44 | `IMG_7b` | { |
| 7 | image_08.jpg | 14:21:03 | `IMG_7469` | ti |
| 8 | image_20.jpg | 14:24:58 | `IMG_6d` | m |
| 9 | image_05.jpg | 14:29:12 | `IMG_696e` | in |
| 10 | image_16.jpg | 14:31:47 | `IMG_67` | g |
| 11 | image_02.jpg | 14:35:20 | `IMG_2d69` | -i |
| 12 | image_18.jpg | 14:39:06 | `IMG_73` | s |
| 13 | image_09.jpg | 14:42:55 | `IMG_2d65` | -e |
| 14 | image_13.jpg | 14:46:13 | `IMG_76` | v |
| 15 | image_04.jpg | 14:51:38 | `IMG_6572` | er |
| 16 | image_15.jpg | 14:55:04 | `IMG_79` | y |
| 17 | image_10.jpg | 14:59:27 | `IMG_7468` | th |
| 18 | image_06.jpg | 15:03:10 | `IMG_69` | i |
| 19 | image_17.jpg | 15:07:42 | `IMG_6e67` | ng |
| 20 | image_11.jpg | 15:12:29 | `IMG_7d` | } |

The decoded fragments formed a complete flag when concatenated in timestamp order.

### Reproduce the Decryption with Python

To verify the process programmatically, I created:

```text
decode_exfiltration.py
```

The script used ExifTool's JSON output to reconstruct the message.

```python
import subprocess
import json

result = subprocess.run(
    [
        "exiftool",
        "-json",
        "-FileName",
        "-DateTimeOriginal",
        "-ImageUniqueID",
        "exfiltration/",
    ],
    capture_output=True,
    text=True,
    check=True,
)

images = json.loads(result.stdout)

images.sort(key=lambda image: image["DateTimeOriginal"])

decoded = []

for image in images:
    image_id = image["ImageUniqueID"]
    hex_value = image_id.removeprefix("IMG_")

    text = bytes.fromhex(hex_value).decode("ascii")
    decoded.append(text)

    print(
        f'{image["FileName"]:<15} '
        f'{image["DateTimeOriginal"]} '
        f'{hex_value:<6} -> {text}'
    )

message = "".join(decoded)

print("\nDecoded message:", message)
```

The script automated the same operations established during manual analysis.

### Verify the Recovered Flag

I executed the decoder:

```bash
python decode_exfiltration.py
```

**Verified output:**

```text
Decoded message: deadface{timing-is-everything}
```

The recovered plaintext matched the expected DEADFACE flag structure.

The presence of the complete flag demonstrated that the timestamp ordering and hexadecimal interpretation were consistent with the intended encoding scheme.

---

## 4. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Recovered Flag:**

`deadface{timing-is-everything}`

**Local Verification:** ✅ Successfully reproduced using Python.

**Points Available:** 75.

**Competition Submission Status:** Not yet confirmed.

</details>

---

## 5. References

- [ExifTool — Official Documentation](https://exiftool.org/)
- [Python — Standard Library Documentation](https://docs.python.org/3/library/)
- [Pillow — Python Imaging Library](https://pillow.readthedocs.io/)
- [NIST — Digital Forensics Resources](https://www.nist.gov/itl/ssd/software-quality-group/computer-forensics-tool-testing-program-cftt)

---

[← Return to Steganography Collection](../README.md)
