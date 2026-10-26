# Digital Design — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Cryptography |
| Challenge | Digital Design |
| Points | 100 |
| Author | @G2Gh0st |
| Platform | GhostTown |
| Technique | Boolean Logic / Binary Decoding / A1Z26 |
| Status | ✅ Solved |

---

## 1. Challenge Description & Objective

### Scenario

Investigators found a post by lilith on Ghosttown that looks like some new crazy encryption scheme but they can't make heads or tails of it. Find the post with a large binary string and an alphabetic string that contains the letters A, O, N, and X and see if you can break the code.

### Objective

Determine how the binary sequence and its alphabetic companion work together, decode the resulting message, and construct the flag.

**Required flag format:**

`deadface{FLAGTEXT}`

---

## 2. Technical Knowledge & Concepts

### 2.1 What Is Binary?

Binary is a base-2 numbering system that represents information using two digits:

- `0`
- `1`

Computers use binary to represent data and perform logical operations.

For example:

```text
Binary: 00101
Decimal: 5
```

In this challenge, binary groups served as inputs to a series of logical operations.

### 2.2 What Is Boolean Logic?

Boolean logic is a system of logical operations that evaluates values such as true and false, commonly represented as `1` and `0`.

Digital circuits use Boolean logic to process binary information.

The challenge used four operations:

| Symbol | Operation | Description |
|---|---|---|
| A | AND | Produces 1 only when both inputs are 1 |
| O | OR | Produces 1 when at least one input is 1 |
| N | NOT | Inverts a single input bit |
| X | XOR | Produces 1 when the two inputs differ |

### 2.3 AND Gate

An AND gate requires two inputs.

It produces 1 only when both inputs are 1.

| A | B | A AND B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 0 |
| 1 | 0 | 0 |
| 1 | 1 | 1 |

### 2.4 OR Gate

An OR gate requires two inputs.

It produces 1 when either input is 1.

| A | B | A OR B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 1 |

### 2.5 NOT Gate

A NOT gate requires only one input.

It reverses the bit.

| Input | NOT |
|---|---|
| 0 | 1 |
| 1 | 0 |

### 2.6 XOR Gate

An XOR gate, or Exclusive OR, requires two inputs.

It produces 1 when the inputs differ.

| A | B | A XOR B |
|---|---|---|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

### 2.7 What Is A1Z26?

A1Z26 is an alphabetic encoding system that assigns a numerical value to each letter.

Examples:

| Number | Letter |
|---|---|
| 1 | A |
| 2 | B |
| 3 | C |
| 9 | I |
| 14 | N |
| 25 | Y |
| 26 | Z |

In this challenge, each set of five logical operations produced a five-bit binary number.

The resulting binary value was converted into decimal and interpreted as an alphabetic position using A1Z26.

---

## 3. Investigation Methodology

#### Objective

Locate the GhostTown post and identify the information required to decode the message.

The relevant post was published by `lilith`.

The author described the message as a form of homemade encryption and provided two sequences.

**Original binary message:**

```text
001100000 10110011 001110101 100111010 010101000 101111101 100101100 00001100 10001010 100000010 0100001110
```

**Original companion string:**

```text
XXONO NXNOA OAOON NXXAO NAAXO XANXO ANOXA ANOXN NONOX ANXAX AOAAX
```

The post stated that both sequences were required and that their spaces were deliberate.

This suggested that corresponding groups should be analyzed together.

---

### Phase 2 — Prepare the Investigation Workspace

Organize the evidence separately from the GitHub documentation repository.

The investigation directory was:

```text
~/CTF-Evidence/Cryptography/Digital-Design/
```

Navigate to the folder:

```bash
cd ~/CTF-Evidence/Cryptography/Digital-Design
```

The binary sequence was saved as `binary.txt`, and the companion string was saved as `companion.txt`.

A Python script was used to compare the group structures:

```python
from pathlib import Path

binary = Path("binary.txt").read_text().split()
companion = Path("companion.txt").read_text().split()

print(f"Binary groups: {len(binary)}")
print(f"Companion groups: {len(companion)}")
print()

for i, (bits, letters) in enumerate(zip(binary, companion), 1):
    print(f"{i:2}. {bits:12} ({len(bits)} bits)  {letters}")
```

**Observed results:**

| Group | Binary input | Length | Companion |
|---|---|---:|---|
| 1 | `001100000` | 9 | `XXONO` |
| 2 | `10110011` | 8 | `NXNOA` |
| 3 | `001110101` | 9 | `OAOON` |
| 4 | `100111010` | 9 | `NXXAO` |
| 5 | `010101000` | 9 | `NAAXO` |
| 6 | `101111101` | 9 | `XANXO` |
| 7 | `100101100` | 9 | `ANOXA` |
| 8 | `00001100` | 8 | `ANOXN` |
| 9 | `10001010` | 8 | `NONOX` |
| 10 | `100000010` | 9 | `ANXAX` |
| 11 | `0100001110` | 10 | `AOAAX` |

**Finding:** Both sequences contained 11 groups.

Every companion group contained exactly five characters, while the binary groups contained between eight and ten bits.

---

### Phase 3 — Identify the Logical Operations

Determine the relationship between the alphabetic companion strings and the binary inputs.

The four letters corresponded to common digital logic operations:

```text
A = AND
O = OR
N = NOT
X = XOR
```

AND, OR, and XOR require two input bits.

NOT requires only one input bit.

Since each companion group contains five operations, the expected number of input bits can be calculated as:

\[
\text{Required Bits} = 10 - (\text{Number of NOT Operations})
\]

For example:

| Companion | NOT operations | Required bits | Actual bits |
|---|---:|---:|---:|
| `XXONO` | 1 | 9 | 9 |
| `NXNOA` | 2 | 8 | 8 |
| `AOAAX` | 0 | 10 | 10 |

The expected and actual lengths matched.

**Conclusion:** The companion string identifies five logic gates, and the corresponding binary group supplies their inputs in sequence.

Each gate generates one output bit, producing a five-bit result.

---

### Phase 4 — Decode the First Group

Test the logic-gate interpretation and determine how the output represents a character.

**Binary:** `001100000`

**Companion:** `XXONO`

#### Step 1: Divide the Binary into Gate Inputs

```text
X    X    O   N   O
00 | 11 | 00 | 0 | 00
```

#### Step 2: Apply the Logic Gates

| Gate | Input | Operation | Output |
|---|---|---|---|
| XOR | `00` | 0 XOR 0 | 0 |
| XOR | `11` | 1 XOR 1 | 0 |
| OR | `00` | 0 OR 0 | 0 |
| NOT | `0` | NOT 0 | 1 |
| OR | `00` | 0 OR 0 | 0 |

**Combined output:**

```text
00010
```

#### Step 3: Convert Binary to Decimal

\[
00010_2=2_{10}
\]

Using A1Z26:

```text
2 = B
```

**Recovered letter:** B

---

### Phase 5 — Decode the Second Group

**Binary:** `10110011`

**Companion:** `NXNOA`

#### Input Grouping

```text
N   X   N   O   A
1 | 01 | 1 | 00 | 11
```

#### Gate Calculations

| Gate | Input | Output |
|---|---|---|
| NOT | `1` | 0 |
| XOR | `01` | 1 |
| NOT | `1` | 0 |
| OR | `00` | 0 |
| AND | `11` | 1 |

**Combined output:**

```text
01001
```

\[
01001_2=9_{10}
\]

Using A1Z26:

```text
9 = I
```

**Recovered letter:** I

The message now began:

```text
BI
```

---

### Phase 6 — Decode All Remaining Groups

Apply the same gate-processing method to the remaining nine binary groups.

Each group was processed in the following order:

1. Identify its five logic gates.
2. Divide the binary input according to each gate's required number of inputs.
3. Calculate the output of each operation.
4. Combine the five output bits.
5. Convert the resulting binary value into decimal.
6. Convert the decimal value into a letter using A1Z26.

#### Complete Decoding Table

| Group | Companion | Gate outputs | Decimal | Letter |
|---|---|---|---:|---|
| 1 | `XXONO` | `00010` | 2 | B |
| 2 | `NXNOA` | `01001` | 9 | I |
| 3 | `OAOON` | `01110` | 14 | N |
| 4 | `NXXAO` | `00001` | 1 | A |
| 5 | `NAAXO` | `10010` | 18 | R |
| 6 | `XANXO` | `11001` | 25 | Y |
| 7 | `ANOXA` | `01100` | 12 | L |
| 8 | `ANOXN` | `01111` | 15 | O |
| 9 | `NONOX` | `00111` | 7 | G |
| 10 | `ANXAX` | `01001` | 9 | I |
| 11 | `AOAAX` | `00011` | 3 | C |

#### Recovered Letters

```text
B I N A R Y L O G I C
```

#### Final Decoded Message

```text
BINARYLOGIC
```

The decoded text was consistent with the challenge's digital logic theme.

---

### Phase 7 — Flag Construction & Verification

The challenge required:

```text
deadface{FLAGTEXT}
```

The recovered text was inserted into the flag format and submitted to the DEADFACE CTF platform.

**Validation result:** Accepted.

**Challenge status:** Solved.

---

## Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{BINARYLOGIC}`

**Status:** Accepted — Challenge Solved

</details>

---
