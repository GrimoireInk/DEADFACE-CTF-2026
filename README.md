# DEADFACE CTF 2026

### Cybersecurity Challenge Write-Ups & Technical Investigations

Welcome to my DEADFACE CTF 2026 documentation repository!

This repository documents my experience participating in my first DEADFACE Capture the Flag competition, including challenge investigations, technical methodologies, security vulnerabilities, and lessons learned.

My goal is to build a comprehensive cybersecurity knowledge base while strengthening my skills in digital forensics, reverse engineering, mobile security, cryptography, OSINT, and security research.

---

## Competition Overview

| Metric | Result |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Final Placement | **262nd / 1,048 teams** |
| Points Earned | **4,406** |
| Participation | First DEADFACE CTF |
| Documentation Status | In Progress |

---

## Challenge Collections

### Aurora Patient Zero

A series of challenges investigating potential information exposure within the fictional Aurora Health Patient Portal iOS application.

**Challenges Documented:**

| Challenge | Status | Write-Up |
|---|---|---|
| Info Leaks | ✅ Solved | [View Investigation](challenges/aurora-patient-zero/info-leaks/README.md) |
| Down the Rabbit Hole | ✅ Solved | [View Investigation](challenges/aurora-patient-zero/down-the-rabbit-hole/README.md) |
| SQL Skillz | ✅ Solved | [View Investigation](sql-skillz/README.md) |
| Back to Basics | ✅ Solved | [View Investi](back-to-basics/README.md) |

**[Explore the Aurora Patient Zero Collection →](challenges/aurora-patient-zero/README.md)**

### Miscellaneous

| Challenge | Write-up |
|---|---|
| AI Usage Policy | [View Write-up](challenges/miscellaneous/ai-usage-policy/README.md) |
| Starter | [View Write-up](challenges/miscellaneous/starter/README.md) |

**[View Miscellaneous Collection](challenges/miscellaneous/README.md)**

### Cryptography

Cryptography challenges involving classical ciphers, encoding, XOR encryption, substitution ciphers, Boolean logic, and chess-based cryptography.

Six challenges were solved during DEADFACE CTF 2026. Chess Key is included as a post-CTF reference study, with the published solution credited to TheMusk3teers.

| Challenge | Points | Status | Write-up |
|---|---:|---|---|
| Hail Caesar | 5 | ✅ Solved | [View Write-up](challenges/cryptography/hail-caesar/README.md) |
| DECODE-DLE | 25 | ✅ Solved | [View Write-up](challenges/cryptography/decode-dle/README.md) |
| X Marks the Spot | 30 | ✅ Solved | [View Write-up](challenges/cryptography/x-marks-the-spot/README.md) |
| Infrequent Emojis | 50 | ✅ Solved | [View Write-up](challenges/cryptography/infrequent-emojis/README.md) |
| Second Shift | 90 | ✅ Solved | [View Write-up](challenges/cryptography/second-shift/README.md) |
| Digital Design | 100 | ✅ Solved | [View Write-up](challenges/cryptography/digital-design/README.md) |
| Chess Key | 200 available | 📖 Unsolved — Post-CTF Reference Study | [View Write-up](challenges/cryptography/chess-key/README.md) |

**[View Cryptography Collection](challenges/cryptography/README.md)**

---

## Technical Skills & Concepts

The challenges in this repository involve developing practical experience with:

- **Mobile Application Security:** iOS IPA structure, application bundles, configuration files, and static analysis.
- **Digital Forensics:** File examination, metadata analysis, evidence handling, and information recovery.
- **Information Disclosure:** Identifying exposed application data and sensitive metadata.
- **Encoding & Data Formats:** Base64, JSON, binary property lists, and structured application resources.
- **Command-Line Investigation:** macOS Terminal, file enumeration, string extraction, and targeted searches.
- **Security Documentation:** Reproducible investigations, evidence-based findings, and technical reporting.
- **Cryptography & Cryptanalysis:** Caesar ciphers, ROT13, monoalphabetic substitution, known-plaintext attacks, XOR keystream analysis, and Vigenère cipher concepts.
- **Python Automation & Data Analysis:** PGN parsing with `python-chess`, processing compressed chess archives using `zstandard`, automated game filtering, coordinate extraction, and ciphertext analysis.
- **Open-Source Intelligence (OSINT):** Researching public chess game databases, correlating information from online discussions, and evaluating external evidence to investigate cryptographic challenges.

Additional skills and techniques will be documented as new challenges are completed.

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| macOS Terminal | Command-line investigation |
| ExifTool | File and image metadata examination |
| `unzip` | Application archive extraction |
| `file` | File format identification |
| `find` | File and directory enumeration |
| `grep` | Targeted text searches |
| `strings` | Extracting printable data from binaries |
| `plutil` | Apple property-list analysis |
| `base64` | Decoding encoded application data |
| VS Code | Documentation and Markdown editing |
| Git & GitHub | Version control and portfolio management |
| Python 3 | Investigation scripting, data parsing, and cryptographic analysis |
| python-chess | PGN parsing, chess move reconstruction, and coordinate extraction |
| zstandard | Processing compressed `.zst` chess game archives |
| Lichess PGN Archives | Public chess game records used for OSINT research |

---

## Documentation Approach

Each challenge write-up is structured to include:

1. **Challenge Information:** Original scenario, category, and objective.
2. **Technical Knowledge & Concepts:** Explanations of unfamiliar technologies and security principles.
3. **Investigation Methodology:** Commands, techniques, and reasoning used during the investigation.
4. **Findings & Evidence:** Verified observations and supporting artifacts.
5. **Security Implications:** Vulnerabilities, exposure risks, and defensive considerations.
6. **Lessons Learned:** Skills developed and improvements for future investigations.
7. **Challenge Flag:** A collapsible spoiler section containing the confirmed flag.

### Post-CTF Reference Studies

Challenges not solved during the competition may be documented as post-CTF reference studies. These entries clearly distinguish my independent investigation from externally sourced solutions, credit original authors, and do not count unsolved challenges toward my competition points or completed challenge totals.

Both successful and unsuccessful techniques may be documented when they contribute to understanding the investigation.

---

## Disclaimer

This repository contains documentation from authorized Capture the Flag challenges conducted during DEADFACE CTF 2026.

All Aurora Health patient records referenced in the challenge environment are fictional.

The techniques discussed are presented for educational cybersecurity research and skill development.

Challenge flags may be placed inside collapsible spoiler sections. These sections help prevent accidental spoilers but do not conceal flags from readers of the raw Markdown.

---

## Future Development

As I continue documenting DEADFACE CTF 2026, this repository will expand to include additional challenge collections, investigation notes, forensic techniques, and supporting technical resources.

The long-term objective is to maintain a searchable, reproducible record of my cybersecurity learning and CTF experience.
