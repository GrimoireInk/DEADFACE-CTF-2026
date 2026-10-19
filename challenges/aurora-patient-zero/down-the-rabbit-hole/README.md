# Down the Rabbit Hole — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| CTF | DEADFACE CTF 2026 |
| Challenge Set | Aurora Patient Zero |
| Challenge | Down the Rabbit Hole |
| Category | Mobile Security |
| Status | Solved |
| Application | Aurora Patient Portal |
| Platform | iOS |
| Evidence | AuroraPatientPortal.ipa.zip |
| Analysis Environment | macOS Terminal |

## 1. Challenge Description & Objective

### Scenario

For this task, you need to continue your examination of the Aurora Patient Portal application.

We know the application relies heavily on a unique Patient ID number for cross-reference throughout the application. Continue your analysis of the app environment to see if you can find a file hiding the Patient ID.

Hint: no emulator or debugging is required for this challenge.

The application evidence is the same archive provided for the preceding [Info Leaks](../info-leaks/README.md) challenge.

### Objective

Locate the hidden Patient ID within the provided application files and submit it using the required format:

`deadface{ID}`

---

## 2. Technical Knowledge & Concepts

### 2.1 What Is File Metadata?

Metadata is information stored about or within a file that describes its properties or contents.

For image files, metadata can include:

- File creation and modification information
- Image dimensions and resolution
- Camera or device information
- GPS coordinates
- Image descriptions and identifiers
- Software information

Metadata is not necessarily visible when viewing the image itself.

**Relevance to this challenge:**

The patient identifier was discovered in the metadata of a JPEG image contained within the Aurora Patient Portal application.

### 2.2 What Is EXIF?

EXIF (Exchangeable Image File Format) is a standard used to store information associated with digital images.

EXIF metadata commonly includes camera information, image timestamps, and photographic settings.

Although EXIF is a common form of image metadata, it is not the only one.

### 2.3 What Is XMP?

XMP (Extensible Metadata Platform) is a metadata framework used to store descriptive and structured information in files.

XMP can contain properties such as identifiers, descriptions, keywords, creator information, and other application-specific metadata.

**Relevance to this challenge:**

ExifTool identified an `Identifier` property associated with the image's embedded metadata. The metadata record contained the fictional patient's ID.

The displayed tag name alone does not establish its exact metadata namespace. That would require examining the tag's group information.

### 2.4 What Is ExifTool?

ExifTool is a command-line utility designed to read, write, and analyze metadata stored in many file formats.

It can inspect JPEG images, PNG files, PDFs, and numerous other media formats.

Example:

```bash
exiftool image.jpg
```

This displays metadata discovered within the specified image.

To display only a particular field:

```bash
exiftool -Identifier image.jpg
```

**Relevance to this challenge:**

ExifTool was used to inspect the chest X-ray image and recover its embedded patient identifier without modifying the image.

### 2.5 What Is Information Disclosure Through Metadata?

Metadata can unintentionally expose information that the creator did not intend to share.

For example, an image might appear anonymous but still contain a person's name, location, or unique identifier within its metadata.

Removing identifying text from an image's visible pixels does not necessarily remove identifying information stored in the file.

**Security significance:**

Applications that distribute medical images should ensure sensitive metadata is removed or appropriately protected when it is not required for the intended functionality.

---

## 3. Investigation Methodology

### Phase 1 — Evidence Preparation

#### Objective

Prepare a dedicated workspace and extract the same application archive used during the Info Leaks investigation.

#### Step 1: Create the Investigation Directory

```bash
cd ~/CTF-Evidence/Aurora-Patient-Zero

mkdir -p Down-the-Rabbit-Hole

cd Down-the-Rabbit-Hole
```

**Command explanation:**

- `cd` changes the current working directory.
- `mkdir` creates a new directory.
- `-p` creates missing parent directories if necessary.

The dedicated folder keeps the current investigation separate from the previous challenge.

#### Step 2: Copy the Original Evidence

```bash
cp ../Info-Leaks/AuroraPatientPortal.ipa.zip .
```

**Command explanation:**

- `cp` copies a file.
- `../Info-Leaks/` references the previous challenge's evidence directory.
- `.` represents the current directory.

This allowed the same original application archive to be reused without downloading it again.

#### Step 3: Inspect the Archive

```bash
unzip -l AuroraPatientPortal.ipa.zip | head -n 25
```

**Command explanation:**

- `unzip -l` lists archive contents without extracting them.
- `|` sends the output of one command to another.
- `head -n 25` limits the displayed output to 25 lines.

**Actual output:**

```text
Archive: AuroraPatientPortal.ipa.zip

Length     Date        Time    Name
---------  ----------  -----   ----
2598224    05-31-2026  19:52   AuroraPatientPortal.ipa
```

**Finding:** 

The supplied ZIP contains a single iOS application archive named `AuroraPatientPortal.ipa`.

#### Step 4: Extract the Application

First, extract the IPA from the outer ZIP:

```bash
unzip AuroraPatientPortal.ipa.zip
```

Create a directory for the application contents:

```bash
mkdir -p extracted-ipa
```

Extract the IPA:

```bash
unzip AuroraPatientPortal.ipa -d extracted-ipa
```

This produced the application bundle under:

`extracted-ipa/Payload/aurora-patient-portal.app`

---

### Phase 2 — Hidden File Investigation

Determine whether the Patient ID was stored within a hidden file in the application's directory structure.

#### Step 1: Search for Hidden Files

I initially interpreted the reference to a file hiding the Patient ID as a possible reference to a Unix-style hidden file.

On macOS and other Unix-like operating systems, filenames beginning with a period are conventionally hidden in standard directory listings.

**Command used:**

```bash
find extracted-ipa/Payload/aurora-patient-portal.app \
  -type f -name ".*"
```

**Command explanation:**

- `find` recursively searches the specified directory.
- `-type f` limits matches to regular files.
- `-name ".*"` matches filenames beginning with a period.

**Actual result:**

The command returned no matching files.

**Analysis:**

The search did not identify any dotfiles within the extracted application bundle.

The challenge's wording could also refer to information concealed within the contents or metadata of an ordinary file.

I therefore shifted the investigation toward examining potentially relevant packaged resources.

---

### Phase 3 — Image Metadata Examination

Investigate whether an application resource contains an embedded Patient ID.

#### Step 1: Identify a Relevant Image

During the previous Info Leaks investigation, application file enumeration revealed a JPEG image named:

`aurora_chest_xray.jpg`

Because the application is a fictional healthcare portal, and the challenge asks for an identifier associated with patient data, the medical image was selected for metadata examination.

#### Step 2: Examine the Chest X-Ray Metadata

```bash
exiftool \
  extracted-ipa/Payload/aurora-patient-portal.app/aurora_chest_xray.jpg
```

**Command explanation:**

- `exiftool` reads metadata from supported files.
- The path identifies the JPEG image being examined.
- No metadata modification options were used.

**Relevant actual output:**

```text
File Name          : aurora_chest_xray.jpg
File Type          : JPEG
File Size          : 309 kB
Image Width        : 1375
Image Height       : 1144
XMP Toolkit        : Image::ExifTool 13.55
Identifier         : PID-847261
```

#### Step 3: Interpret the Metadata

The important discovery was:

```text
Identifier : PID-847261
```

The `Identifier` field contained a value using the prefix `PID-`, followed by a six-digit number.

This was consistent with the challenge's description of a unique Patient ID.

**Raw metadata identifier:** `PID-847261`

**Patient ID:** `847261`

The remaining metadata primarily described the image's technical characteristics and was not necessary to identify the Patient ID.

#### Step 4: Isolate the Relevant Metadata

A more focused ExifTool command can retrieve the identifier without displaying every metadata field:

```bash
exiftool -Identifier \
  extracted-ipa/Payload/aurora-patient-portal.app/aurora_chest_xray.jpg
```

This command is included as a streamlined verification method. The initial discovery was made using the full ExifTool output.

---

### Phase 4 — Flag Construction and Validation

Convert the recovered Patient ID into the required challenge flag format and validate it.

**Required format:**

`deadface{ID}`

The image metadata contained:

`PID-847261`

The first candidate preserved the complete identifier, including the `PID-` prefix. However, the challenge did not accept that candidate.

After removing the prefix and retaining only the numerical ID, the flag was accepted.

**Confirmed finding:**

The required ID was the six-digit numerical portion of the metadata identifier.

**Challenge status:** Solved.

---

## 4. Findings & Evidence

| Evidence | Finding | Significance |
|---|---|---|
| AuroraPatientPortal.ipa.zip | Contains AuroraPatientPortal.ipa | Original application evidence |
| Application bundle | Contains aurora_chest_xray.jpg | Relevant packaged image |
| Hidden-file search | No dotfiles located | Initial hypothesis unsupported |
| Image metadata | Identifier field: PID-847261 | Patient ID recovered |
| Flag submission | Numeric ID accepted | Challenge solution confirmed |

### Root Cause

The supplied application package contained a medical image with a recoverable patient identifier embedded in its metadata.

The identifier could be extracted through static file analysis without running the application or authenticating to a patient account.

---

## 5. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{847261}`

**Status:** Accepted — Challenge Solved

</details>
