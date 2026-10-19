
# Info Leaks — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| CTF | DEADFACE CTF 2026 |
| Challenge Set | Aurora Patient Zero |
| Challenge | Info Leaks |
| Category | Mobile Security |
| Points | 25 |
| Author | @G2Gh0st |
| Application | Aurora Patient Portal |
| Platform | iOS |
| Evidence | AuroraPatientPortal.ipa.zip |
| Status | Completed |

---

## 1. Challenge Description & Objective

### Scenario

Aurora Health has been dealing with a wave of highly targeted phishing attacks against its patients. The messages are convincing, contain accurate patient information, and are resulting in an alarming number of successful compromises.

Investigators have identified one common link among the victims: every affected patient recently installed Aurora Health's new Patient Portal iOS application. Security analysts suspect the app may be exposing sensitive information that DEADFACE is using to craft its phishing campaigns. We have been given a copy of the application for analysis. Fully evaluate the app and determine what information is being leaked so we can truly identify the source of the exposure before more patients fall victim to the attacks.

Hint: no emulator or debugger is required for this challenge.

### Objective

Identify the first name, last name, and date of birth
of the patient who last used the supplied application
version.

**Required flag format:**

`deadface{firstname-lastname-YYYY-MM-DD}`

**Challenge hint:**

No emulator or debugger is required.

---

## 2. Technical Knowledge & Concepts

### 2.1 What Is an IPA File?

An IPA is an archive format used to package iOS
applications for distribution.

IPA files use the ZIP archive format, allowing their
contents to be inspected without executing the application.

### 2.2 What Is an iOS Application Bundle?

An application bundle is a directory containing
the application's executable, configuration files,
and packaged resources.

On iOS, application bundles normally have a
`.app` extension.

### 2.3 What Is Static Analysis?

Static analysis is the examination of software
without executing it.

For this challenge, static analysis allows us
to inspect the application package for information
that may have been unintentionally included.

### 2.4 What Is Info.plist?

Info.plist is an Apple property list used to store
application configuration information.

It may include the application's name, identifier,
version, and executable filename.

### 2.5 What Is Sensitive Information Exposure?

Sensitive information exposure occurs when
confidential information becomes accessible
to an unauthorized party.

Patient names and dates of birth are examples of
personal information that require appropriate protection.


### 2.6 What Is Base64 Encoding?

Base64 is a binary-to-text encoding scheme that
represents data using printable ASCII characters.

It is commonly used to transport binary data in
text-based formats.

Base64 is not encryption because decoding does not
require a secret key.

**Relevance to this challenge:**

Patient information was stored in `Info.plist` as
Base64-encoded JSON.

The encoding initially obscured the values from
ordinary text searches, but the original information
could be recovered using `base64 -D`.

### 2.7 What Is JSON?

JSON (JavaScript Object Notation) is a lightweight
format used to represent structured data.

It commonly stores information as key-value pairs.

**Relevance to this challenge:**

After decoding the Base64 string, the recovered
data was identified as a JSON object containing
patient information.

---

## 3. Investigation Methodology

### Phase 1 — Evidence Acquisition and Verification

The challenge provided an archive named
`AuroraPatientPortal.ipa.zip`.

Because the analysis was performed on macOS, the ZIP
archive was extracted automatically by macOS
Archive Utility when opened through Finder.

This produced the file:

`AuroraPatientPortal.ipa`

The IPA was subsequently examined using macOS Terminal.

### Evidence Integrity

A SHA-256 hash was calculated for the extracted IPA:

```bash
shasum -a 256 AuroraPatientPortal.ipa
```

**Calculated SHA-256:**

`142e70d4f1db8eaca3f7e88cf446d442724472379196bcada7ad56531ce3b8db`

This hash did not match the reference supplied by
the challenge.

However, the reference appears to correspond to the
original `.ipa.zip` archive rather than the extracted
`.ipa` file.

Because these are different files, their hashes
are not expected to match.

### Phase 2 — IPA Extraction and File Inspection

Extract the Aurora Patient Portal iOS application and
inspect its internal files to identify potentially
relevant artifacts for further analysis.

#### Step 1: Identify the IPA File Format

After obtaining `AuroraPatientPortal.ipa`, I used
the macOS `file` utility to identify its actual format.

```bash
file AuroraPatientPortal.ipa
```

**Command explanation:**

- `file` identifies a file's format by examining its contents.
- The filename specifies the file being examined.

**Actual output:** AuroraPatientPortal.ipa: Zip archive data, at least v1.0 to extract, compression method=store

**Finding:**

The IPA uses the ZIP archive format, meaning its
contents can be examined and extracted using standard
ZIP utilities without running the iOS application.

#### Step 2: Examine the IPA Contents

Before extracting the application, I listed the
archive's contents:

```bash
unzip -l AuroraPatientPortal.ipa
```

**Command explanation:**

- `unzip` is used to work with ZIP archives.
- `-l` lists the archive contents without extracting them.

#### Step 3: Extract the IPA

I created a directory to store the extracted files:

```bash
mkdir -p extracted-zip
```

Then extracted the IPA:

```bash
unzip AuroraPatientPortal.ipa -d extracted-zip
```

**Command explanation:**

- `mkdir -p` creates the extraction directory.
- `unzip` extracts the archive.
- `-d` specifies the destination for extracted files.

#### Step 4: Locate the Application Bundle

I searched for an iOS application bundle:

```bash
find extracted-zip -type d -name "*.app"
```

**Command explanation:**

- `find` searches the specified directory recursively.
- `-type d` limits the search to directories.
- `-name "*.app"` matches application bundles ending
  in `.app`.

**Actual output:** extracted-zip/Payload/aurora-patient-portal.app

**Finding:**

The extracted IPA contains an application bundle
named `aurora-patient-portal.app`, located inside
the `Payload` directory.

#### Step 5: Examine the Application Files

To list the contents of the application bundle:

```bash
ls -lah extracted-zip/Payload/aurora-patient-portal.app
```

To enumerate the packaged files recursively:

```bash
find extracted-zip/Payload/aurora-patient-portal.app -type f
```

**Command explanation:**

- `ls -lah` displays directory contents, including
  hidden entries, file sizes, and permissions.
- `find ... -type f` lists regular files recursively.

**Actual findings:**

| File | Description |
|---|---|
| Info.plist | Application configuration |
| Aurora Health Mobile | Compiled iOS executable |
| aurora_meds.sqlite | SQLite database |
| aurora_chest_xray.jpg | JPEG image |
| Assets.car | Compiled application asset catalog |
| embedded.mobileprovision | Provisioning information |
| PkgInfo | Application package metadata |
| AppIcon files | Application icons |
| _CodeSignature/CodeResources | Code-signing resources |

#### Step 6: Identify File Formats

To determine the actual formats of the extracted
application files, I used:

```bash
find extracted-zip/Payload/aurora-patient-portal.app \
  -type f -exec file {} \;
```

**Command explanation:**

- `-exec` runs a command for each matching file.
- `file` identifies the file's likely format.
- `{}` represents the current file.
- `\;` terminates the command passed to `-exec`.

**Key results:**

| Artifact | Identified format |
|---|---|
| Info.plist | Apple binary property list |
| Aurora Health Mobile | Mach-O 64-bit executable, ARM64 |
| aurora_meds.sqlite | SQLite 3.x database |
| aurora_chest_xray.jpg | JPEG image with EXIF metadata |
| Assets.car | Apple asset catalog |

### Phase 2 Conclusion

The Aurora Patient Portal IPA was successfully
extracted and its application bundle identified.

The examination revealed configuration files,
an executable, an embedded database, and image
resources.

These findings provide potential areas for further
analysis, but no patient identity or date of birth
has yet been established.

### Phase 3 — Targeted Patient Information Discovery

Identify information within the Aurora Patient Portal
application that could reveal the name and date of birth
of the patient who last used this version of the app.

Following the initial examination of the iOS application
bundle, I focused on the compiled executable because it
could contain embedded strings related to patient data,
application functionality, or locally stored information.

#### Step 1: Search for Patient-Related Information

I began by searching the extracted application files
for keywords potentially associated with patient identity.

**Command used:**

```bash
grep -RIlE \
  'patient|date.?of.?birth|dob|last.?user|last.?login' \
  extracted-zip/Payload/aurora-patient-portal.app
```

**Command explanation:**

- `grep` searches files for matching text patterns.
- `-R` searches directories recursively.
- `-I` skips files identified as binary.
- `-l` returns matching filenames instead of their contents.
- `-E` enables extended regular expressions.
- `|` within the quoted expression represents OR.

**Actual result:**

The command returned no matching filenames.

**Analysis:**

The initial search did not identify files containing
the specified text patterns.

Since some application resources are stored as compiled
binaries, I proceeded to examine the executable directly.

---

#### Step 2: Extract Readable Strings from the Executable

During Phase 2, I identified the application's main
executable as:

`Aurora Health Mobile`

The `file` utility previously identified this executable
as a Mach-O 64-bit ARM64 binary.

I used `strings` to search for embedded references to
patient information.

**Command used:**

```bash
strings -a \
  "extracted-zip/Payload/aurora-patient-portal.app/Aurora Health Mobile" \
  | grep -iE 'patient|name|birth|dob|user|login' \
  | head -n 30
```

**Command explanation:**

- `strings -a` extracts printable character sequences
  from the executable.
- `grep -iE` filters results using a case-insensitive
  extended regular expression.
- `head -n 30` displays the first 30 matching lines.

**Actual output:**

```text
PatientBootstrap
patientName
dateOfBirth
patient
imagingStudyFileName
name
fileName
AuroraPatientBootstrap
Demo chart for Patient:
CTF patient companion
Patient Cache Realm
patient-summary-cache-v2
```

**Analysis:**

The executable contains references to `patientName`
and `dateOfBirth`, directly corresponding to the two
types of information requested by the challenge.

The `PatientBootstrap` identifier suggests the presence
of a component associated with initializing or loading
patient-related information.

Additionally, `patient-summary-cache-v2` suggests that
the application may implement a patient-data caching
mechanism.

---

#### Step 3: Investigate PatientBootstrap References

To examine additional strings surrounding the
`PatientBootstrap` identifier, I used:

```bash
strings -a \
  "extracted-zip/Payload/aurora-patient-portal.app/Aurora Health Mobile" \
  | grep -i -C 12 'PatientBootstrap'
```

**Command explanation:**

- `strings -a` extracts printable strings from the
  executable.
- `grep -i` performs a case-insensitive search.
- `-C 12` displays 12 lines before and after each match.

**Purpose:**

Examine surrounding strings to identify possible
references to application components, patient data,
or the location of stored information.

**Relevant findings:**

```text
Aurora_Health_Mobile
Aurora_Health_MobileApp
PatientBootstrap
AuroraAppointment
WellnessCheck
ContentView
CodingKeys
DiagnosticScreen
ImagingStudyRow
AuroraPatientBootstrap
Demo chart for Patient:
All information is fictional and not suitable
for actual healthcare decisions.
CTF patient companion
gw-westlake-int-03
Patient Cache Realm
patient-summary-cache-v2
0 pending records
Imaging Link Mode
local bundled asset
```

**Analysis:**

Several notable identifiers were discovered:

| Identifier | Potential significance |
|---|---|
| PatientBootstrap | Patient initialization or data-loading component |
| AuroraPatientBootstrap | Potentially related patient bootstrap type |
| CodingKeys | Common Swift identifier associated with encoding and decoding data |
| DiagnosticScreen | Application diagnostic interface |
| Patient Cache Realm | Possible patient caching or storage reference |
| patient-summary-cache-v2 | Potential cache identifier |
| gw-westlake-int-03 | Unidentified application-specific identifier |
| local bundled asset | Indicates functionality referencing packaged resources |

---

#### Phase 3 — Current Findings

Identify the patient information exposed within the
Aurora Patient Portal application and determine how
it can be recovered without executing the application.

The challenge requires the patient's first name,
last name, and date of birth.

#### Step 1: Search the Application for Patient Information

I initially performed a recursive keyword search
through the extracted application bundle.

**Command used:**

```bash
grep -RIlE \
  'patient|date.?of.?birth|dob|last.?user|last.?login' \
  extracted-zip/Payload/aurora-patient-portal.app
```

**Command explanation:**

- `grep` searches for matching text patterns.
- `-R` recursively searches directories.
- `-I` skips files treated as binary.
- `-l` displays matching filenames rather than content.
- `-E` enables extended regular expressions.

**Actual result:**

The command returned no matching filenames.

**Analysis:**

The initial search did not identify any readable files
containing the selected keywords.

However, since the application contains compiled binary
files, I could not conclude that patient information
was absent.

#### Step 2: Examine Strings in the Executable

I next examined the application's main executable,
`Aurora Health Mobile`, for readable strings
associated with patient information.

**Command used:**

```bash
strings -a \
  "extracted-zip/Payload/aurora-patient-portal.app/Aurora Health Mobile" \
  | grep -iE 'patient|name|birth|dob|user|login' \
  | head -n 30
```

**Command explanation:**

- `strings -a` extracts printable character sequences
  from the executable.
- `grep -iE` searches using case-insensitive patterns.
- `head -n 30` limits displayed results.

**Relevant output:**

```text
PatientBootstrap
patientName
dateOfBirth
patient
imagingStudyFileName
name
fileName
AuroraPatientBootstrap
Demo chart for Patient:
CTF patient companion
Patient Cache Realm
patient-summary-cache-v2
```

**Analysis:**

Two identifiers directly corresponded to the
challenge objective:

- `patientName`
- `dateOfBirth`

The `AuroraPatientBootstrap` identifier also appeared
potentially relevant because it suggested a component
associated with initializing patient information.

However, these strings did not reveal the actual
patient identity.

#### Step 3: Investigate the Patient Bootstrap Reference

To determine whether additional information was stored
near the `PatientBootstrap` identifier, I examined
surrounding strings.

**Command used:**

```bash
strings -a \
  "extracted-zip/Payload/aurora-patient-portal.app/Aurora Health Mobile" \
  | grep -i -C 12 'PatientBootstrap'
```

**Relevant findings:**

```text
PatientBootstrap
AuroraPatientBootstrap
CodingKeys
DiagnosticScreen
Demo chart for Patient:
CTF patient companion
Patient Cache Realm
patient-summary-cache-v2
```

**Analysis:**

The additional strings suggested that the application
contained patient-related models and potentially
cached patient information.

However, no actual patient name or date of birth
was recovered through this examination.

The investigation therefore shifted toward identifying
where the `AuroraPatientBootstrap` reference was stored.

#### Step 4: Locate AuroraPatientBootstrap

I searched the application bundle for files containing
the exact string `AuroraPatientBootstrap`.

**Command used:**

```bash
grep -Ral -F 'AuroraPatientBootstrap' \
  extracted-zip/Payload/aurora-patient-portal.app
```

**Command explanation:**

- `-R` searches recursively.
- `-a` treats binary files as text for matching.
- `-l` displays only matching filenames.
- `-F` performs a literal string search.

**Actual output:**

```text
extracted-zip/Payload/aurora-patient-portal.app/Aurora Health Mobile
extracted-zip/Payload/aurora-patient-portal.app/Info.plist
```

**Analysis:**

The identifier appeared in both the executable
and the application's `Info.plist` configuration file.

Since `Info.plist` can contain developer-defined
configuration properties, I investigated this file
as a possible location for embedded patient data.

#### Step 5: Identify the Property Type

I used `plutil` to determine the data type associated
with the `AuroraPatientBootstrap` property.

**Command used:**

```bash
plutil -type AuroraPatientBootstrap \
  "extracted-zip/Payload/aurora-patient-portal.app/Info.plist"
```

**Actual output:**

```text
string
```

**Finding:**

The `AuroraPatientBootstrap` property contains a
string value.

This indicated that the information could potentially
be extracted directly from the property list.

#### Step 6: Extract the Property Value

I initially attempted to extract the property as JSON:

```bash
plutil -extract AuroraPatientBootstrap json -o - \
  "extracted-zip/Payload/aurora-patient-portal.app/Info.plist"
```

**Actual result:**

```text
Invalid object in plist for JSON format
```

Since the property was confirmed to contain a string,
I changed the extraction format to `raw`.

**Command used:**

```bash
plutil -extract AuroraPatientBootstrap raw -o - \
  "extracted-zip/Payload/aurora-patient-portal.app/Info.plist"
```

**Command explanation:**

- `-extract AuroraPatientBootstrap` selects the property.
- `raw` outputs its string value directly.
- `-o -` sends the result to standard output.

**Actual result:**

```text
eyJwYXRpZW50TmFtZSI6Ik1hcmEgVm9zcyIsImRhdGVPZkJpcnRoIjoiMTk4OC0wNC0xNyIsImNsaW5pYyI6Ildlc3RsYWtlIn0=
```

**Analysis:**

The extracted string appeared to use Base64 encoding.

Base64 converts binary or textual information into
a representation using printable ASCII characters.

#### Step 7: Decode the Embedded Patient Information

To decode the recovered value, I used macOS's
built-in `base64` utility.

**Command used:**

```bash
plutil -extract AuroraPatientBootstrap raw -o - \
  "extracted-zip/Payload/aurora-patient-portal.app/Info.plist" \
  | base64 -D
```

**Command explanation:**

- `plutil` retrieves the encoded string.
- `|` passes the output to the next command.
- `base64 -D` decodes Base64 input on macOS.

The result was a JSON object containing the
following properties:

```json
{
  "patientName": "[REDACTED]",
  "dateOfBirth": "[REDACTED]",
  "clinic": "Westlake"
}
```

**Finding:**

The encoded application property contained the
patient's full name, date of birth, and clinic.

These values could be recovered without launching
the application, authenticating as a patient, or
using an emulator or debugger.

---

## 🚩 Challenge Flag

<details>
<summary><strong>Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{mara-voss-1988-04-17}`

**Status:** Correct — Accepted by DEADFACE CTF 2026

</details>
