# Infrequent Emojis — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Cryptography |
| Challenge | Infrequent Emojis |
| Points | 50 |
| Technique | Monoalphabetic Substitution Cipher / Pattern Analysis |
| Platform | DoomScroll |
| Source | https://doomscroll.dctf.io/ |
| Status | Solved |

---

## 1. Challenge Description & Objective

### Scenario

While DoomScrolling, our intel team came across a weird coded post with emojis all over it. One of our experts claims that the patterns point to it still being English but we don't know what it means! Can you figure it out?

### Objective

Identify the emoji-encoded message, determine the substitution system, and recover the original plaintext.

**Required flag format:**

`deadface{flag_text}`

---

## 2. Technical Knowledge & Concepts

### 2.1 What Is a Substitution Cipher?

A substitution cipher replaces characters in a plaintext message with other characters or symbols according to a predefined mapping.

For example:

```text
Plaintext:  HELLO
Ciphertext: ★☀☂☂◆
```

In this example, each symbol consistently represents one letter.

If the same substitution mapping is used throughout the message, recurring plaintext letters produce recurring ciphertext symbols.

### 2.2 What Is a Monoalphabetic Substitution Cipher?

A monoalphabetic substitution cipher uses one consistent mapping between plaintext letters and ciphertext symbols.

For example, if 🕯️ represents `E`, every occurrence of 🕯️ should decode to `E`.

This consistency allows an analyst to use repeated symbols, word patterns, and known plaintext to reconstruct the mapping.

### 2.3 What Is Frequency Analysis?

Frequency analysis involves examining how often letters or symbols appear in a message.

In English, letters such as E, T, A, O, I, and N appear relatively frequently.

A substitution cipher may disguise the letters themselves, but often preserves statistical properties of the original language.

For this investigation, repeated-symbol patterns and contextual word recognition were more important than numerical frequency counts.

### 2.4 What Is Known-Plaintext Analysis?

Known-plaintext analysis uses information about an expected portion of the original message to infer a substitution or encryption mapping.

Because DEADFACE CTF flags normally begin with `deadface{`, the emojis preceding the opening brace provided a strong starting point.

Recognizing this known structure allowed multiple emoji-to-letter substitutions to be identified immediately.

---

## 3. Investigation Methodology

### Phase 1 — Locate the Suspicious Post

Find the emoji-encoded message on DoomScroll.

The challenge provided the following website:

https://doomscroll.dctf.io/

During the investigation, a suspicious post associated with the username `spookyboi` was identified.

The post contained an unusual sequence of emojis, underscores, and curly braces.

The curly braces were particularly relevant because they resembled the expected DEADFACE flag structure.

#### Step 1: Prepare the Evidence Directory

```bash
cd ~/CTF-Evidence/Cryptography

mkdir -p Infrequent-Emojis

cd Infrequent-Emojis
```

#### Step 2: Preserve the Ciphertext

The emoji sequence was copied into a local text file named `emojis.txt`.

```bash
nano emojis.txt
```

This preserved the original emoji characters and their order for analysis.

#### Original Ciphertext

```text
🪦 🕯️ 💀 🪦 🩸 💀 ⚰️ 🕯️ { 🦉 🕸️ 🕯️ 💀 🌙 🕯️ _ ☣️ 🔮 🧪 _ 🖤 🌩️ ☣️ _ ⛓️ 🧪 _ 🦇 🕯️ 🔥 _ 🌙 ☣️ 💀 🦇 🪦 💀 🔮 🪦 }
```

---

### Phase 2 — Identify the Flag Prefix

Determine whether the initial emojis represented the standard DEADFACE CTF flag prefix.

The first eight emojis were:

```text
🪦 🕯️ 💀 🪦 🩸 💀 ⚰️ 🕯️
```

The repeated-symbol pattern matched the eight letters in `deadface`.

| Emoji | Plaintext letter |
|---|---|
| 🪦 | D |
| 🕯️ | E |
| 💀 | A |
| 🩸 | F |
| ⚰️ | C |

Applying the mapping:

```text
🪦 🕯️ 💀 🪦 🩸 💀 ⚰️ 🕯️
 D  E  A  D  F  A  C  E
```

**Finding:** The ciphertext begins with the expected `deadface` prefix.

This established a useful partial substitution alphabet.

---

### Phase 3 — Decode the First Word

Use the known mappings to identify the first word inside the braces.

The first emoji group was:

```text
🦉 🕸️ 🕯️ 💀 🌙 🕯️
```

Using the previously identified letters:

```text
? ? E A ? E
```

The six-letter word **PLEASE** matched the pattern and the message's likely context.

| Emoji | Letter |
|---|---|
| 🦉 | P |
| 🕸️ | L |
| 🕯️ | E |
| 💀 | A |
| 🌙 | S |

The message now began:

```text
deadface{please_..._..._.._..._........}
```

---

### Phase 4 — Decode the Final Word

Use a longer partially decoded word to recover additional substitutions.

The last emoji group was:

```text
🌙 ☣️ 💀 🦇 🪦 💀 🔮 🪦
```

The known mappings produced:

```text
S ? A ? D A ? D
```

The word **STANDARD** fit the eight-letter pattern.

This revealed three additional mappings:

| Emoji | Letter |
|---|---|
| ☣️ | T |
| 🦇 | N |
| 🔮 | R |

The final word was identified as:

```text
STANDARD
```

These new substitutions could now be applied to the remaining words.

---

### Phase 5 — Decode the Remaining Words

#### Step 1: Recover the Second Word

The second emoji group was:

```text
☣️ 🔮 🧪
```

Using the known mappings:

```text
T R ?
```

The word **TRY** fit the pattern and sentence structure.

This established:

```text
🧪 = Y
```

#### Step 2: Recover the Third Word

The third emoji group was:

```text
🖤 🌩️ ☣️
```

The known mappings gave:

```text
? ? T
```

The word **OUT** fit the emerging instruction, "Please try out..."

This established:

```text
🖤 = O
🌩️ = U
```

#### Step 3: Recover the Fourth Word

The fourth emoji group was:

```text
⛓️ 🧪
```

The mapping for 🧪 produced:

```text
? Y
```

The word **MY** fit the sentence.

This established:

```text
⛓️ = M
```

#### Step 4: Recover the Fifth Word

The fifth emoji group was:

```text
🦇 🕯️ 🔥
```

Using the known mappings:

```text
N E ?
```

The word **NEW** completed the sentence.

This established:

```text
🔥 = W
```

---

### Phase 6 — Reconstruct the Complete Message

Applying the recovered substitutions produced:

```text
deadface{please_try_out_my_new_standard}
```

The readable message was:

**Please try out my new standard.**

The original underscores were preserved as word separators in the flag.

#### Complete Emoji Substitution Table

| Emoji | Letter |
|---|---|
| 🪦 | D |
| 🕯️ | E |
| 💀 | A |
| 🩸 | F |
| ⚰️ | C |
| 🦉 | P |
| 🕸️ | L |
| 🌙 | S |
| ☣️ | T |
| 🔮 | R |
| 🧪 | Y |
| 🖤 | O |
| 🌩️ | U |
| ⛓️ | M |
| 🦇 | N |
| 🔥 | W |

Each repeated emoji consistently represented the same plaintext letter.

---

## 4. Findings & Evidence

| Evidence | Finding | Significance |
|---|---|---|
| DoomScroll post | Emoji-encoded message | Original challenge evidence |
| Curly braces | Standard flag structure | Suggested known plaintext |
| Initial eight emojis | `deadface` | Established initial substitutions |
| First word | `please` | Expanded the mapping |
| Final word | `standard` | Revealed additional letters |
| Remaining words | `try_out_my_new` | Completed the message |
| Flag submission | Accepted | Confirmed the solution |

---

## 5. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{please_try_out_my_new_standard}`

**Status:** Accepted — Challenge Solved

</details>
