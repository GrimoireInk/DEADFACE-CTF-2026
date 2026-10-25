# X Marks the Spot — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Cryptography |
| Challenge | X Marks the Spot |
| Points | 30 |
| Author | @syyntax |
| Technique | Hexadecimal Decoding / Single-Byte XOR |
| XOR Key | `0x44` |
| Status | Solved |

---

## 1. Challenge Description & Objective

### Scenario

Zero Trace's network sensors picked up this outbound packet from a host registered to Spectre Technology. The payload is hex-encoded. Decrypt it to find what spookyboi is hiding.

### Objective

Decode the hexadecimal payload, identify any additional encryption, and recover the hidden flag.

**Required flag format:** `deadface{flag-text}`

---

## 2. Technical Knowledge & Concepts

### 2.1 What Is Hexadecimal Encoding?

Hexadecimal is a base-16 number system that uses the digits `0–9` and the letters `A–F`.

Each hexadecimal digit represents four bits. Two hexadecimal digits represent one byte.

For example:

```text
Hexadecimal: 48 65 6c 6c 6f
ASCII:        H  e  l  l  o
```

Hexadecimal is commonly used to display binary data in packet captures, memory dumps, and forensic evidence.

**Important distinction:** Hexadecimal is an encoding format, not an encryption algorithm.

### 2.2 What Is XOR?

XOR (Exclusive OR) is a bitwise operation that produces 1 when two input bits differ and 0 when they are the same.

| A | B | A XOR B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

XOR is reversible when the same key is applied twice.

\[
(P \oplus K) \oplus K = P
\]

Where:

- \(P\) = Plaintext
- \(K\) = XOR key
- \(\oplus\) = XOR operation

This property makes XOR useful in encryption algorithms, although repeating a single fixed byte as a key is not secure.

### 2.3 What Is Single-Byte XOR Encryption?

Single-byte XOR encryption applies the same byte value to every byte of plaintext.

For example:

```python
encrypted_byte = plaintext_byte ^ key
```

Decryption uses the same operation:

```python
decrypted_byte = encrypted_byte ^ key
```

Because the key is only one byte, there are 256 possible key values.

This makes single-byte XOR vulnerable to exhaustive key searches and plaintext analysis.

### 2.4 What Is Known-Plaintext Analysis?

Known-plaintext analysis uses information about probable plaintext to infer an encryption key.

For example, if an encrypted byte is suspected to represent an ASCII space, XORing it with `0x20` may reveal a candidate key.

In this investigation, repeated occurrences of `0x64` suggested that the byte might represent spaces.

This led to the candidate key:

```text
0x64 XOR 0x20 = 0x44
```

The candidate was then tested against additional ciphertext.

---

## 3. Investigation Methodology

Preserve the intercepted hexadecimal payload in a dedicated investigation workspace.

#### Step 1: Navigate to the Evidence Directory

```bash
cd ~/CTF-Evidence/Cryptography/X-Marks-the-Spot
```

#### Step 2: Save the Hexadecimal Payload

The payload was saved as `payload.txt`.

```bash
nano payload.txt
```

**Original hexadecimal payload:**

```text
02362b2964302c2164372c25202b3337642b226417342127303621641021272c2a2b282b233d7e6420212520222527213f3c2b361b751b263d30771b70301b701b3075297739
```

#### Step 3: Inspect the Payload

```bash
wc -c payload.txt
cat payload.txt
```

This allowed the hexadecimal payload to be inspected before decoding.

---

### Phase 2 — Convert Hexadecimal into Bytes

Decode the hexadecimal representation to recover the underlying byte sequence.

The following command was used:

```bash
xxd -r -p payload.txt > decoded.bin
```

**Command explanation:**

- `xxd` converts data between hexadecimal and binary representations.
- `-r` performs reverse conversion.
- `-p` specifies plain hexadecimal input.
- `>` redirects output into a file.

The decoded bytes were examined using:

```bash
xxd decoded.bin
```

The byte sequence did not immediately produce readable English text.

**Finding:** An additional transformation was likely required.

---

### Phase 3 — Identify the XOR Key

Determine whether the payload used XOR encryption and identify a candidate key.

The challenge title, *X Marks the Spot*, suggested XOR.

The ciphertext contained repeated occurrences of the hexadecimal byte `64`.

Because spaces occur frequently in English text, the investigation tested whether `0x64` represented an encrypted space.

ASCII space:

```text
0x20
```

Candidate XOR key:

```text
0x64 XOR 0x20 = 0x44
```

**Candidate key:** `0x44` (decimal 68)

#### Step 1: Test the First Ciphertext Byte

The first byte was:

```text
0x02
```

Applying XOR:

```text
0x02 XOR 0x44 = 0x46
```

ASCII `0x46` represents the uppercase letter `F`.

#### Step 2: Test Additional Bytes

| Ciphertext | XOR Key | Plaintext |
|---|---|---|
| `02` | `44` | F |
| `36` | `44` | r |
| `2b` | `44` | o |
| `29` | `44` | m |

**Recovered text:** `From`

The candidate key successfully produced a recognizable English word.

#### Step 3: Verify the Following Words

The next ciphertext segment was:

```text
64 30 2c 21
```

Applying the same key produced:

```text
 the
```

The following seven-letter word was decrypted using Python:

```bash
python3 -c "print(''.join(chr(int(x, 16) ^ 0x44) for x in '37 2c 25 20 2b 33 37'.split()))"
```

**Output:**

```text
shadows
```

At this point, the message began:

```text
From the shadows
```

This provided strong evidence that `0x44` was the correct key.

---

### Phase 4 — Decrypt the Complete Payload

Apply the identified XOR key to the entire ciphertext.

The following Python script was used:

```python
from pathlib import Path

hex_data = Path("payload.txt").read_text().strip()

ciphertext = bytes.fromhex(hex_data)
key = 0x44

plaintext = bytes(byte ^ key for byte in ciphertext)

print("Decrypted message:")
print(plaintext.decode("utf-8"))
```

#### Code Explanation

| Python code | Purpose |
|---|---|
| `Path(...).read_text()` | Reads the hexadecimal payload |
| `bytes.fromhex()` | Converts hexadecimal text into bytes |
| `key = 0x44` | Defines the candidate XOR key |
| `byte ^ key` | Applies XOR to each byte |
| `bytes(...)` | Combines the resulting bytes |
| `.decode("utf-8")` | Converts the result to readable text |

#### Decrypted Output

```text
From the shadows of Spectre Technology: deadface{xor_1_byt3_4t_4_t1m3}
```

The plaintext contained a flag in the expected DEADFACE CTF format.

---

### Phase 5 — Flag Verification

The recovered flag was submitted to the DEADFACE CTF platform.

**Validation result:** Accepted.

**Challenge status:** Solved.

---

## 4. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{xor_1_byt3_4t_4_t1m3}`

**Status:** Accepted — Challenge Solved

</details>
