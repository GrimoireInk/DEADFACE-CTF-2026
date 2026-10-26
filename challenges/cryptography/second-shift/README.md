# Second Shift — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Cryptography |
| Challenge | Second Shift |
| Points | 90 |
| Author | @syyntax |
| Difficulty | Intermediate |
| Technique | Known-Plaintext Attack / XOR Keystream Reuse |
| Status | ✅ Solved |

---

## 1. Challenge Description & Objective

### Scenario

ZeroTrace recovered several files from a DEADFACE relay server shortly before the host went offline. The relay appears to have been used by mort1cia to exchange operational messages with other DEADFACE members.

One routine transmission was preserved both before and after processing by the relay. A second outbound transmission was recovered only in its processed form.

Determine what DEADFACE was preparing to do next and recover the flag.

### Objective

Investigate the relay's encryption behavior, recover the contents of the second transmission, and identify the hidden flag.

**Required flag format:**

`deadface{flag-text}`

---

## 2. Technical Knowledge & Concepts

### 2.1 What Is a Known-Plaintext Attack?

A known-plaintext attack is a cryptanalytic technique in which an investigator possesses both the plaintext and corresponding ciphertext for at least one message.

By comparing these values, the investigator may discover weaknesses in the encryption process or recover information useful for decrypting other messages.

In this challenge, the known plaintext and ciphertext were:

- `relay-heartbeat.json`
- `relay-heartbeat.enc`

The recovered information was then applied to:

- `relay-tasking.enc`

### 2.2 What Is XOR?

XOR, or Exclusive OR, is a bitwise operation that produces 1 when two input bits differ and 0 when they are equal.

| A | B | A XOR B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

XOR is reversible because applying the same value twice restores the original data.

\[
(P \oplus K) \oplus K = P
\]

Where:

- \(P\) = Plaintext
- \(K\) = Keystream
- \(\oplus\) = XOR operation

### 2.3 What Is a Keystream?

A keystream is a sequence of values combined with plaintext to produce ciphertext.

For an XOR-based stream encryption process:

\[
C=P\oplus K
\]

Decryption reverses the transformation:

\[
P=C\oplus K
\]

The security of stream encryption depends on generating and using the keystream correctly.

### 2.4 What Is Keystream Reuse?

Keystream reuse occurs when the same keystream is applied to multiple plaintext messages.

Suppose two messages are encrypted using the same keystream:

\[
C_1=P_1\oplus K
\]

\[
C_2=P_2\oplus K
\]

If the first plaintext and ciphertext are known, the keystream can be recovered:

\[
K=P_1\oplus C_1
\]

That keystream can then decrypt corresponding bytes of the second ciphertext:

\[
P_2=C_2\oplus K
\]

Substituting the recovered keystream gives:

\[
P_2=C_2\oplus P_1\oplus C_1
\]

This relationship was the key to solving Second Shift.

### 2.5 What Is CTR Mode?

Counter mode, commonly abbreviated CTR, is a mode of operation that turns a block cipher into a stream-like encryption mechanism.

A counter value, combined with a nonce, is processed using an encryption key to generate keystream blocks.

Those blocks are XORed with plaintext to produce ciphertext.

The nonce must not be reused with the same key in a way that repeats the counter inputs.

If the same key, nonce, and counter sequence are reused, the resulting keystream may repeat, compromising confidentiality.

**Important distinction:** The challenge's recovered flag references CTR nonce reuse. Our investigation directly demonstrated keystream reuse, but we did not independently verify the relay's underlying cipher implementation.

---

## 3. Investigation Methodology

### Phase 1 — Evidence Preparation

Prepare a dedicated investigation workspace and identify the recovered relay files.

#### Step 1: Navigate to the Evidence Directory

```bash
cd ~/CTF-Evidence/Cryptography/Second-Shift
```

#### Step 2: Examine the Extracted Evidence

The challenge archive had been extracted into a directory named `secondshift`.

```bash
ls -lah secondshift/
```

**Observed files:**

```text
relay-heartbeat.enc
relay-heartbeat.json
relay-tasking.enc
```

#### Step 3: Identify the File Types

```bash
find secondshift -type f -exec file {} \;
```

**Actual output:**

```text
secondshift/relay-tasking.enc: data
secondshift/relay-heartbeat.enc: data
secondshift/relay-heartbeat.json: JSON data
```

The `.enc` extension suggested that the files contained encrypted or otherwise transformed data.

The JSON file provided the readable message needed for known-plaintext analysis.

---

### Phase 2 — Examine the Known Plaintext

Read the original heartbeat transmission and inspect its encrypted counterpart.

#### Step 1: Read the Heartbeat JSON

```bash
cat secondshift/relay-heartbeat.json
```

**Actual output:**

```json
{"type":"heartbeat","host":"relay-07","user":"mort1cia","status":"ok","queue":3,"channel":"ops","timestamp":"2026-10-16T23:41:08Z","message":"No anomalies detected. Continuing scheduled relay checks.","build":"relay-agent-3.8.14","region":"us-south"}
```

The JSON represented a routine relay heartbeat containing operational information, including:

- Host: `relay-07`
- User: `mort1cia`
- Status: `ok`
- Channel: `ops`
- Build: `relay-agent-3.8.14`
- Region: `us-south`

#### Step 2: Examine the Encrypted Heartbeat

```bash
xxd -l 128 secondshift/relay-heartbeat.enc
```

The encrypted file began with:

```text
16 b7 a1 33 9f ab 35 6a ef 40 4a 77 ef 97 f0 88
39 19 57 12 a5 2e fd ed c8 10 19 6a fa 16 e2 10
```

The bytes did not resemble the original JSON plaintext.

**Finding:** A binary transformation had been applied to the heartbeat message.

---

### Phase 3 — Investigate File Lengths

Determine why the original heartbeat was one byte larger than its encrypted counterpart.

#### Step 1: Compare File Sizes

```bash
wc -c secondshift/relay-heartbeat.json secondshift/relay-heartbeat.enc secondshift/relay-tasking.enc
```

**Actual output:**

```text
251 secondshift/relay-heartbeat.json
250 secondshift/relay-heartbeat.enc
220 secondshift/relay-tasking.enc
721 total
```

The plaintext JSON was 251 bytes, while its encrypted counterpart was 250 bytes.

#### Step 2: Examine the End of the JSON

```bash
tail -c 32 secondshift/relay-heartbeat.json | xxd
```

The final bytes included:

```text
7d 0a
```

These correspond to:

- `0x7d` — Closing curly brace `}`
- `0x0a` — Newline character

**Finding:** The original JSON contained a trailing newline.

Removing this newline produced a 250-byte plaintext matching the ciphertext length.

This was consistent with the relay processing the JSON without its final newline.

---

### Phase 4 — Test the XOR Hypothesis

Determine whether the heartbeat was encrypted using a repeating XOR key.

The first investigation calculated a candidate XOR keystream by comparing the known plaintext and ciphertext.

```python
from pathlib import Path

plain = Path("secondshift/relay-heartbeat.json").read_bytes()
cipher = Path("secondshift/relay-heartbeat.enc").read_bytes()

candidate_key = bytes(
    p ^ c for p, c in zip(plain, cipher)
)

print(candidate_key[:32].hex(" "))
```

**Recovered candidate bytes:**

```text
6d 95 d5 4a ef ce 17 50 cd 28 2f 16 9d e3 92 ed
58 6d 75 3e 87 46 92 9e bc 32 23 48 88 73 8e 71
```

No obvious short repeating pattern was visible.

#### Step 1: Test Repeating Key Lengths

A Python script was used to test possible repeating XOR key lengths from 1 to 32 bytes.

```python
from pathlib import Path

plain = Path("secondshift/relay-heartbeat.json").read_bytes()
cipher = Path("secondshift/relay-heartbeat.enc").read_bytes()

plain = plain.rstrip(b"\r\n")

candidate_key = bytes(
    p ^ c for p, c in zip(plain, cipher)
)

print("Plaintext length:", len(plain))
print("Ciphertext length:", len(cipher))

print("\nTesting repeating XOR key lengths:")

for length in range(1, 33):
    matches = sum(
        candidate_key[i] == candidate_key[i % length]
        for i in range(len(candidate_key))
    )

    percentage = matches / len(candidate_key) * 100

    if percentage > 30:
        print(f"Key length {length}: {percentage:.1f}% matching")
```

**Actual output:**

```text
Plaintext length: 250
Ciphertext length: 250

Testing repeating XOR key lengths:
```

No candidate key length produced a match above the 30% reporting threshold.

**Conclusion:** A simple repeating XOR key of length 1–32 bytes was not supported by the test.

This did not rule out XOR-based encryption using a nonrepeating keystream.

---

### Phase 5 — Compare the Encrypted Transmissions

Determine whether the heartbeat and tasking ciphertexts shared structural similarities.

#### Step 1: Examine Their Initial Bytes

```bash
xxd -l 32 secondshift/relay-heartbeat.enc
xxd -l 32 secondshift/relay-tasking.enc
```

**Heartbeat ciphertext:**

```text
16 b7 a1 33 9f ab 35 6a ef 40 4a 77 ef 97 f0 88
```

**Tasking ciphertext:**

```text
16 b7 a1 33 9f ab 35 6a ef 5c 4e 65 f6 8a fc 8a
```

The first nine bytes of both ciphertexts were identical:

```text
16 b7 a1 33 9f ab 35 6a ef
```

The shared prefix suggested that both messages might use the same initial keystream.

However, identical ciphertext prefixes alone do not prove keystream reuse because identical plaintext prefixes could also produce identical ciphertext under certain conditions.

A direct decryption test was necessary.

---

### Phase 6 — Test Keystream Reuse

Determine whether the candidate keystream recovered from the heartbeat could decrypt the tasking transmission.

#### Step 1: Recover the Keystream

Using the known heartbeat plaintext and ciphertext:

```python
keystream = bytes(
    p ^ c for p, c in zip(plain1, cipher1)
)
```

#### Step 2: Test the First 64 Bytes

The following Python script tested the candidate keystream against the beginning of the tasking ciphertext.

```python
from pathlib import Path

folder = Path("secondshift")

plain1 = (folder / "relay-heartbeat.json").read_bytes().rstrip(b"\r\n")
cipher1 = (folder / "relay-heartbeat.enc").read_bytes()
cipher2 = (folder / "relay-tasking.enc").read_bytes()

key_stream = bytes(
    p ^ c for p, c in zip(plain1, cipher1)
)

test = bytes(
    c ^ k for c, k in zip(cipher2[:64], key_stream[:64])
)

print("Recovered candidate plaintext (first 64 bytes):")
print(repr(test))

print("\nReadable interpretation:")
print(test.decode("utf-8", errors="replace"))
```

#### Actual Output

```text
Recovered candidate plaintext (first 64 bytes):
b'{"type":"tasking","target":"aurora-vpn","action":"stage-access",'

Readable interpretation:
{"type":"tasking","target":"aurora-vpn","action":"stage-access",
```

**Finding:** The recovered bytes formed a valid beginning of a JSON message.

This strongly supported the hypothesis that both transmissions used the same keystream.

The attack was successful without recovering the original encryption key.

---

### Phase 7 — Decrypt the Complete Tasking Message

Apply the recovered keystream to the entire 220-byte tasking ciphertext.

#### Python Decryption Script

```python
from pathlib import Path

folder = Path("secondshift")

# Load the known plaintext and encrypted messages.
plain1 = (folder / "relay-heartbeat.json").read_bytes().rstrip(b"\r\n")
cipher1 = (folder / "relay-heartbeat.enc").read_bytes()
cipher2 = (folder / "relay-tasking.enc").read_bytes()

# Recover the XOR keystream.
keystream = bytes(
    p ^ c for p, c in zip(plain1, cipher1)
)

# Decrypt the second transmission.
plaintext2 = bytes(
    c ^ k for c, k in zip(cipher2, keystream)
)

print("Recovered Transmission 2:")
print(plaintext2.decode("utf-8"))

# Preserve recovered plaintext.
(folder / "relay-tasking-recovered.json").write_bytes(plaintext2)

print("\nSaved to secondshift/relay-tasking-recovered.json")
```

#### Recovered Transmission

```json
{
  "type": "tasking",
  "target": "aurora-vpn",
  "action": "stage-access",
  "window": "00:30Z",
  "operator": "mort1cia",
  "note": "Use the backup tunnel. Archive bundle under DEADFACE{ctr_nonce_reuse_breaks_confidentiality} before exfil."
}
```

The recovered tasking message revealed operational instructions associated with `mort1cia`.

Most importantly, the message contained the challenge flag.

The decrypted plaintext was also saved as:

```text
secondshift/relay-tasking-recovered.json
```

The original evidence files were not modified.

---

## 7. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{ctr_nonce_reuse_breaks_confidentiality}`

**Status:** Accepted — Challenge Solved

</details>
