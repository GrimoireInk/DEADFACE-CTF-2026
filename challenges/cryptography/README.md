# Cryptography — DEADFACE CTF 2026

## Collection Overview

This collection documents my Cryptography challenges from DEADFACE CTF 2026, including classical ciphers, XOR encryption, substitution analysis, and logical deduction.

Each write-up contains the challenge methodology, evidence, technical explanations, findings, and lessons learned.

## Challenge Progress

| Challenge | Points | Status | Write-up |
|---|---:|---|---|
| Hail Caesar | 5 | ✅ Solved | [View Write-up](hail-caesar/README.md) |
| DECODE-DLE | 25 | ✅ Solved | [View Write-up](decode-dle/README.md) |
| X Marks the Spot | 30 | ✅ Solved | [View Write-up](x-marks-the-spot/README.md) |
| Infrequent Emojis | 50 | ✅ Solved | [View Write-up](infrequent-emojis/README.md) |
| Second Shift | 90 | ✅ Solved | [View Write-up](second-shift/README.md) |
| Digital Design | 100 | ✅ Solved | [View Write-up](digital-design/README.md) |
| Chess Key | 200 | 📖 Unsolved — Post-CTF Reference Study | [View Write-Up](chess-key/README.md) |

## Techniques Explored

### Caesar Cipher & ROT13

[Hail Caesar](hail-caesar/README.md) — Decoding a substitution cipher using ROT13 and the Unix `tr` command.

### Logical Deduction

[DECODE-DLE](decode-dle/README.md) — Solving a three-digit combination through constraint elimination and positional reasoning.

### Hexadecimal Encoding & XOR

[X Marks the Spot](x-marks-the-spot/README.md) — Recovering an XOR-encrypted message from a hexadecimal payload using Python.

### Emoji Substitution

[Infrequent Emojis](infrequent-emojis/README.md) — Decoding a monoalphabetic emoji substitution cipher using pattern recognition and known plaintext.

### Known-Plaintext Attacks & Keystream Reuse

[Second Shift](second-shift/README.md) - Recovered a second encrypted transmission by deriving a shared XOR keystream from a known plaintext/ciphertext pair.

### Boolean Logic & Binary Encoding

[Digital Design](digital-design/README.md) - Decoded a message using binary inputs paired with AND, OR, NOT, and XOR logic gates. Converted each five-bit output into a letter using A1Z26.

### Chess-Based Cryptography & OSINT

[Chess Key](chess-key/README.md) — Post-CTF reference study exploring public chess game records, alphabetic chessboard mappings, and Vigenère cipher decryption. The published solution is credited to TheMusk3teers.

### PGN Parsing & Python Automation

[Chess Key](chess-key/README.md) — Independently analyzed over 163,000 chess game records using Python, `python-chess`, and `zstandard`. Extracted chess coordinates, reconstructed a 64-square alphabetic board, and tested key-generation hypotheses.

## Tools & Technologies

- macOS Terminal
- Python 3
- `tr` and `xxd`
- Text editors
- DoomScroll
- Git & GitHub
