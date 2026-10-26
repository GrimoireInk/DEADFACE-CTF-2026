# 🔐 Cryptography — DEADFACE CTF 2026

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

[View Cryptography Collection](challenges/cryptography/README.md)

**Completed Challenges:** 4

**Confirmed Points:** 110, plus Infrequent Emojis.

## Techniques Explored

### Caesar Cipher & ROT13

[Hail Caesar](hail-caesar/README.md) — Decoding a substitution cipher using ROT13 and the Unix `tr` command.

### Logical Deduction

[DECODE-DLE](decode-dle/README.md) — Solving a three-digit combination through constraint elimination and positional reasoning.

### Hexadecimal Encoding & XOR

[X Marks the Spot](x-marks-the-spot/README.md) — Recovering an XOR-encrypted message from a hexadecimal payload using Python.

### Emoji Substitution

[Infrequent Emojis](infrequent-emojis/README.md) — Decoding a monoalphabetic emoji substitution cipher using pattern recognition and known plaintext.

## Tools & Technologies

- macOS Terminal
- Python 3
- `tr` and `xxd`
- Text editors
- DoomScroll
- Git & GitHub

---
