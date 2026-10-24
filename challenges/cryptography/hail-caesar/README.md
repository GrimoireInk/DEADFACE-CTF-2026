# Hail Caesar — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Cryptography |
| Challenge | Hail Caesar |
| Points | 5 |
| Author | @syyntax |
| Difficulty | Beginner |
| Cipher | ROT13 (Caesar Cipher) |
| Status | Solved |

---

## 1. Challenge Description & Objective

### Scenario

One of the faculty at Night Veil University thought it would be clever to write sensitive information on notes by their computer. This sensitive information includes a very easy-to-decipher password hint. Judging by the drawing of a dog on the photo, we assume the password hint is the name of the individual's dog.

### Objective

Decrypt the password hint and identify the dog's name.

**Required flag format:** `deadface{dog's name}`

---

## 2. Technical Knowledge & Concepts

### 2.1 What Is a Caesar Cipher?

A Caesar cipher is a substitution cipher in which every letter is shifted by a fixed number of positions in the alphabet.

Letters wrap around when a shift reaches the end of the alphabet.

Caesar ciphers are examples of classical symmetric encryption because the same shift value can be used to encrypt and decrypt messages by reversing the direction.

### 2.2 What Is ROT13?

ROT13 stands for **Rotate by 13 Places**.

It is a special type of Caesar cipher that shifts each letter 13 positions through the 26-letter English alphabet.

Because 13 is half of 26, applying ROT13 twice restores the original message.

ROT13 is reversible and does not provide meaningful security against modern attacks.

### 2.3 What Is the `tr` Command?

The Unix `tr` command translates or replaces characters according to two specified character sets.

For example:

```bash
echo "Uryyb" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

Output:

```text
Hello
```

The first character set represents the original alphabet, and the second represents the ROT13 substitution alphabet.

---

## 3. Investigation Methodology

Identify the encrypted password hint in the challenge's supplied evidence.

The evidence was saved in a dedicated investigation folder:

```text
~/CTF-Evidence/Cryptography/Hail-Caesar/
```

The downloaded evidence was available locally as:

```text
Hail Caesar Password Hint.pdf
```

The PDF was opened in macOS Preview for visual examination.

```bash
open "Hail Caesar Password Hint.pdf"
```

The following encrypted message was recovered:

```text
Zl qbt'f anzr vf Srgpu
```

**Finding:** The message appeared to be a substitution cipher containing multiple English-like words.

---

### Phase 2 — Cipher Identification

Determine the encryption technique used to encode the password hint.

The challenge title, **Hail Caesar**, suggested a Caesar cipher.

ROT13 was selected as an initial candidate because it is a common variant of the Caesar cipher.

To test the hypothesis, the first two characters were decoded:

| Encrypted | ROT13 result |
|---|---|
| Z | M |
| l | y |

The decoded text began with:

```text
My
```

This supported testing ROT13 against the complete message.

---

### Phase 3 — Message Decryption

Recover the original plaintext message.

The ciphertext was decoded using the macOS Terminal:

```bash
echo "Zl qbt'f anzr vf Srgpu" | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

#### Decoded Output

```text
My dog's name is Fetch
```

The plaintext identified the faculty member's dog as **Fetch**.

---

### Phase 4 — Flag Construction & Verification

The challenge required the dog's name in the following format:

```text
deadface{dog's name}
```

The recovered name was inserted into the flag format.

**Validation result:** The DEADFACE CTF platform accepted the flag.

**Challenge status:** Solved.

---

## 3. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{Fetch}`

**Status:** Accepted — Challenge Solved

</details>
