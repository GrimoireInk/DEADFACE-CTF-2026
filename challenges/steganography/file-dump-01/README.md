# File Dump 01 — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Steganography |
| Challenge | File Dump 01 |
| Points | 10 |
| Challenge Author | @RP-01? |
| Competition Status | ✅ Solved During CTF |
| Solver | Personal Solve |
| Flag Status | ✅ Confirmed |
| Primary Tool | ExifTool 13.55 |
| Operating System for Reproduction | macOS |
| Techniques | File Enumeration, Document Metadata Analysis, Evidence Correlation |

---

## 1. Challenge Description & Objective

### Scenario

We have intercepted a DEADFACE hacker's laptop and determined he had stolen a man named Daniel's entire identity. For this first challenge, let's identify which DEADFACE member is responsible!

### Objective

Identify the DEADFACE member's alias using information recovered from the provided files.

**Required flag format:**

`deadface{deadface_alias}`

### Evidence Provided

The challenge supplied a downloadable file dump containing documents, images, and other files.

The extracted evidence was stored in:

```text
~/CTF-Evidence/Steganography/File-Dump-01/
└── File Dump Challenge/
```

The directory contained 26 files, including Word documents, Excel spreadsheets, PDFs, and images.

The variety of file types required an initial inventory before selecting a forensic analysis method.

---

## 2. Technical Knowledge & Concepts

### 2.1 Digital Forensic File Enumeration

File enumeration is the process of identifying and cataloging files within an evidence collection.

During a forensic investigation, filenames, extensions, sizes, formats, and timestamps provide useful starting points.

A file's extension does not always accurately represent its underlying structure. File identification tools can examine signatures and other characteristics to determine the actual format.

### 2.2 Document Metadata

Metadata is information stored about a file or its contents.

Microsoft Office documents may contain properties such as:

- Creator
- Last modified by
- Document title
- Subject
- Description
- Creation date
- Modification date

These fields can reveal information about the people, accounts, or systems associated with a document.

Metadata should be treated as investigative evidence rather than definitive proof of a person's real-world identity because these fields can be edited or forged.

### 2.3 Microsoft Office Open XML

Modern Microsoft Office formats, including DOCX and XLSX, use ZIP-based containers containing XML documents and supporting resources.

Their internal metadata can be stored in files such as:

```text
docProps/core.xml
docProps/app.xml
```

This structure allows forensic tools to inspect document properties without opening the file in Microsoft Office.

### 2.4 ExifTool

ExifTool is a command-line utility capable of extracting metadata from many file formats, including Word documents, Excel spreadsheets, PDFs, and images.

The following options were used:

| Option | Function |
|---|---|
| `-r` | Recursively examine files in directories |
| `-a` | Display duplicate tags |
| `-G1` | Identify the metadata group |
| `-s` | Use short metadata tag names |
| `-Creator` | Display creator metadata |
| `-LastModifiedBy` | Display the last recorded editor |
| `-Description` | Display document descriptions |

---

## 3. Investigation Methodology

### Prepare the Investigation Workspace

I organized the extracted evidence in a dedicated directory outside the GitHub repository.

```bash
mkdir -p ~/CTF-Evidence/Steganography/File-Dump-01

cd ~/CTF-Evidence/Steganography/File-Dump-01
```

Keeping working evidence separate from public documentation helps avoid accidentally committing unnecessary files or unrelated information.

### Enumerate the Evidence

I listed the contents of the extracted directory:

```bash
ls -lah "File Dump Challenge/"
```

The directory contained 26 files.

The evidence included:

| Category | Files |
|---|---|
| Word documents | `notes.docx`, `resume.docx`, `reminders.docx`, `budget_notes.docx`, and others |
| Excel spreadsheets | `invoice.xlsx`, `mileage.xlsx` |
| PDF documents | `warranty.pdf`, `bad_furniture_ad.pdf` |
| Images | `profile_pic.png`, `businesscard_1.png`, `office_memory_meme.jpg`, and others |

I then inspected the actual file formats:

```bash
find "File Dump Challenge/" -type f -exec file {} \;
```

The results confirmed valid PNG, JPEG, DOCX, XLSX, and PDF files.

At this stage, there was no verified reason to prioritize image-pixel steganography over document analysis.

### Perform a Metadata Survey

Because the challenge required identifying an individual, I searched the files for metadata related to authorship and identification.

```bash
exiftool -r -a -G1 -s \
  -FileName -FileType -Creator -Author -Artist \
  -Title -Subject -Description -Comment \
  -Keywords -Software \
  "File Dump Challenge/"
```

Several documents contained a recurring creator value:

`ghostwriter404`

The most relevant findings were:

| File | Creator | Title or Description |
|---|---|---|
| `invoice.xlsx` | `ghostwriter404` | Invoice - Hameser |
| `reminders.docx` | `ghostwriter404` | Miscellaneous reminders |
| `resume.docx` | `ghostwriter404` | Daniel Hameser - Resume |
| `notes.docx` | `ghostwriter404` | notes |
| `budget_notes.docx` | Daniel Hameser | Budget Notes |
| `warranty.pdf` | Daniel Hameser | Microwave Warranty |

This established a potential connection between the recurring alias and documents associated with Daniel Hameser.

The evidence made `ghostwriter404` a strong candidate for the DEADFACE member's alias.

However, additional corroboration was needed before treating that interpretation as established.

### Examine Document Authorship

I performed a more targeted metadata analysis on the four documents attributed to `ghostwriter404`.

```bash
exiftool -a -G1 -s \
  -Creator -LastModifiedBy -Title -Description \
  -CreateDate -ModifyDate \
  "File Dump Challenge/invoice.xlsx" \
  "File Dump Challenge/reminders.docx" \
  "File Dump Challenge/resume.docx" \
  "File Dump Challenge/notes.docx"
```

The results revealed a consistent pattern.

| File | Creator | Last Modified By |
|---|---|---|
| `invoice.xlsx` | `ghostwriter404` | thomas canfield |
| `reminders.docx` | `ghostwriter404` | thomas canfield |
| `resume.docx` | `ghostwriter404` | thomas canfield |
| `notes.docx` | `ghostwriter404` | GHOST404-PC |

This provided three related identifiers:

**1. `ghostwriter404`**

The same creator value appeared across four documents.

**2. `thomas canfield`**

Three documents recorded this name as their last modifier.

This suggested a possible association between the alias and an individual using that name, but did not independently prove their identity.

**3. `GHOST404-PC`**

The `notes.docx` file identified this value as its last modifier.

Its similarity to `ghostwriter404` strengthened the relationship between the documents and the suspected alias.

### Examine Additional Metadata

One document contained an additional noteworthy property.

The `reminders.docx` description field included:

```text
backup passphrase: n0rth_st4r (do not lose)
```

This was recorded as a potentially relevant investigative artifact.

However, the passphrase was not needed to determine the answer to File Dump 01, and it was not established to be associated with the challenge's required flag.

No additional decryption or password-recovery steps were necessary for this investigation.

### Identify the DEADFACE Alias

The evidence supported the following relationship:

```text
Documents associated with Daniel Hameser
                 |
                 v
       Repeated Creator Field
            ghostwriter404
                 |
          +------+------+
          |             |
          v             v
    thomas canfield  GHOST404-PC
    LastModifiedBy    LastModifiedBy
    (three files)     (notes.docx)
```

The recurring creator value was the strongest direct candidate for the DEADFACE alias requested by the challenge.

The resulting flag was:

`deadface{ghostwriter404}`

The flag was confirmed as correct.

---

## 4. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{ghostwriter404}`

**Competition Status:** ✅ Solved during DEADFACE CTF 2026.

**Solver:** Personal solve.

**Points:** 10.

</details>

---

## 5. References

- [ExifTool — Official Documentation](https://exiftool.org/)
- [Microsoft — Open XML Formats](https://learn.microsoft.com/en-us/office/open-xml/open-xml-sdk)
- [NIST — Digital Forensics Resources](https://www.nist.gov/itl/ssd/software-quality-group/computer-forensics-tool-testing-program-cftt)

---

[← Return to Steganography Collection](../README.md)
