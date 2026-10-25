# DECODE-DLE — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Cryptography |
| Challenge | DECODE-DLE |
| Points | 25 |
| Author | @G2Gh0st |
| Difficulty | Beginner |
| Technique | Logical Deduction / Constraint Satisfaction |
| Status | Solved |

---

## 1. Challenge Description & Objective

### Scenario

One of the junior DEADFACE members left a number puzzle to decipher a 3-digit number combination. Do you have the skills to solve the puzzle and open the lock? Follow the clues to decipher the 3-digit code needed to "pick" the lock. Enter the code as 3 digits, no spaces or other separating characters.

### Objective

Analyze five clues to determine the correct three-digit combination.

**Required flag format:**

`deadface{###}`

---

## 2. Technical Knowledge & Concepts

### 2.1 What Is Logical Deduction?

Logical deduction is the process of reaching conclusions based on established facts and rules.

In this challenge, each clue places restrictions on which digits can appear in the combination and where those digits may be positioned.

By combining these restrictions, we can eliminate invalid possibilities until only one combination remains.

### 2.2 What Is Constraint Satisfaction?

A constraint satisfaction problem (CSP) involves finding values that satisfy a collection of conditions.

For DECODE-DLE:

- **Variables:** The three positions in the lock combination.
- **Values:** Digits from 0 through 9.
- **Constraints:** The five clues provided in the puzzle.

A valid solution must satisfy every clue simultaneously.

### 2.3 Correct Digits vs. Correct Positions

The puzzle distinguishes between two concepts.

**Correct digit, correct position:** A digit appears in the final combination in the same position as shown in the clue.

**Correct digit, incorrect position:** A digit appears in the combination but occupies a different position.

For example, suppose the combination is `123`.

A clue containing `156` would have one correct digit in the correct position: `1`.

A clue containing `312` would have three correct digits, all in incorrect positions.

Understanding this distinction is essential to solving the puzzle.

### 2.4 Why Verification Matters

A proposed combination is not valid simply because it satisfies one or two clues.

It must satisfy all five.

For this challenge, the statements are interpreted as exact counts. For example, "two numbers are correct but in the wrong place" means exactly two digits are present and both occupy incorrect positions.

---

## 3. Investigation Methodology

### Phase 1 — Evidence Preparation

Obtain the puzzle and examine the supplied numerical clues.

#### Step 1: Navigate to the Investigation Workspace

```bash
cd ~/CTF-Evidence/Cryptography/DECODE-DLE
```

#### Step 2: Verify the Evidence File

```bash
ls -lh
```

**Observed file:**

```text
Decode DLE.pdf
```

The PDF was approximately 43 KB.

#### Step 3: Open the Puzzle

```bash
open "Decode DLE.pdf"
```

The document contained five numerical clues describing the three-digit combination.

---

### Phase 2 — Extract the Puzzle Clues

Record each clue and understand its implications.

The following clues were recovered from the supplied PDF:

| Combination | Clue |
|---|---|
| `863` | One number is correct but in the wrong place |
| `250` | Nothing is correct |
| `168` | Two numbers are correct but in the wrong place |
| `735` | One number is correct and in the right place |
| `687` | Two numbers are correct but in the wrong place |

Each clue must be satisfied by the final combination.

---

### Phase 3 — Eliminate Incorrect Digits

#### Step 1: Analyze the Second Clue

**Clue:** `250`

*Nothing is correct.*

This establishes that the following digits do not appear in the combination:

- `2`
- `5`
- `0`

These digits can be eliminated from subsequent reasoning.

#### Step 2: Analyze the Fourth Clue

**Clue:** `735`

*One number is correct and in the right place.*

Since 5 was already eliminated, two possibilities remain:

1. The digit `7` is correct and occupies position 1, while `3` is absent.
2. The digit `3` is correct and occupies position 2, while `7` is absent.

At this stage, neither possibility has been confirmed.

#### Step 3: Compare the First, Third, and Fifth Clues

**Clue 1:** `863` — Exactly one digit is correct but incorrectly positioned.

**Clue 3:** `168` — Exactly two digits are correct but incorrectly positioned.

**Clue 5:** `687` — Exactly two digits are correct but incorrectly positioned.

The third and fifth clues share the digits `6` and `8`.

Suppose both 6 and 8 were present. That would satisfy the two-digit requirement of both clues, but the first clue would then contain two correct digits rather than one.

Therefore, both 6 and 8 cannot be present simultaneously.

Suppose neither 6 nor 8 were present. The third clue would contain at most one correct digit (`1`), contradicting its requirement for two correct digits.

Therefore, **exactly one of 6 or 8 must be present**.

The third clue then requires `1` to be present.

The fifth clue requires `7` to be present.

Consequently, our combination contains:

- `1`
- `7`
- Exactly one of `6` or `8`

This also establishes that `3` is absent.

Returning to the fourth clue, the correct digit must therefore be `7` in position 1.

**Partial combination:**

```text
7 ? ?
```

---

### Phase 4 — Determine Whether 6 or 8 Is Correct

#### Step 1: Test the Digit 8

Suppose 8 appears in the final combination.

From the first clue:

`863` — 8 cannot occupy position 1.

From the third clue:

`168` — 8 cannot occupy position 3.

That would leave position 2.

However, the fifth clue states:

`687` — Two numbers are correct but in the wrong place.

Since 8 also appears in position 2 of this clue, it cannot occupy position 2 either.

Therefore:

| Position | Can 8 occupy it? |
|---|---|
| Position 1 | No |
| Position 2 | No |
| Position 3 | No |

**Conclusion:** The digit `8` must be eliminated.

Since exactly one of 6 or 8 must appear, the correct digit is `6`.

Our three required digits are now:

```text
7, 1, 6
```

---

### Phase 5 — Determine the Correct Positions

#### Step 1: Position the Digit 7

From the fourth clue, 7 is correct and occupies position 1.

```text
7 ? ?
```

#### Step 2: Position the Digit 6

From the first clue:

`863` — 6 is in position 2, but the clue says it is incorrectly positioned.

Therefore, 6 cannot occupy position 2.

From the fifth clue:

`687` — 6 is in position 1, but the clue says it is incorrectly positioned.

Therefore, 6 cannot occupy position 1.

Only position 3 remains.

```text
7 ? 6
```

#### Step 3: Position the Digit 1

The remaining digit is `1`.

Since positions 1 and 3 are occupied, 1 must be placed in position 2.

**Recovered combination:**

```text
7 1 6
```

---

### Phase 6 — Validate the Combination

Verify the proposed combination against every clue.

**Recovered combination:** `716`

| Clue | Verification | Result |
|---|---|---|
| `863` | Only 6 appears, and it is in a different position | Pass |
| `250` | None of these digits appear | Pass |
| `168` | 1 and 6 appear, both in different positions | Pass |
| `735` | Only 7 appears, and it is correctly positioned | Pass |
| `687` | 6 and 7 appear, both in different positions | Pass |

All five constraints are satisfied.

The combination was submitted in the required flag format and confirmed correct.

**Challenge status:** Solved.

---

## 7. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{716}`

**Status:** Correct — Challenge Solved

</details>
