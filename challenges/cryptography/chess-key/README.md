# Chess Key — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Cryptography |
| Challenge | Chess Key |
| Points Available | 200 |
| Challenge Author | @syyntax |
| Techniques | OSINT, Chess Notation, Vigenère Cipher, Known-Key Decryption |
| Competition Status | ❌ Unsolved |
| Documentation Status | 📖 Post-CTF Reference Study |
| Original Solution | [TheMusk3teers](https://github.com/TheMusk3teers/TheMusk3teers.github.io/blob/68b72eeed6a38d019cc18dee32be736ace8959ec/docs/ctfs/deadface-2026/cryptography/chess-key.md) |

> **Important — Attribution and Competition Status**
>
> I attempted Chess Key during DEADFACE CTF 2026 but did not complete it during the competition.
>
> After the event, I revisited the challenge and conducted additional independent research, including investigating GhostTown, locating chess game records, parsing PGN archives, and reconstructing the provided alphabetic chessboard.
>
> However, I did not independently recover the correct game, decryption method, or flag.
>
> The completed solution documented below is based on the publicly available write-up by **TheMusk3teers**, credited in the References section.
>
> This document combines my own investigation notes with a clearly attributed explanation of the published solution. It is intended for educational purposes and is **not a claim of a competition solve or points earned**.

---

## 1. Challenge Description & Objective

### Scenario

Looks like DEADFACE compromised a machine and left this cryptic calling card:

checkmate. nZ3ipU8_yy3p5_hvyKi0

One of our other analysts also found this image that we believe is tied to this calling card.

### Objective

Identify the relationship between the encrypted calling card, the provided alphabetic chessboard, and the external chess-related information.

**Required flag format:**

`deadface{flag-text}`

### Provided Evidence

- `chesskey.png` — 8×8 alphabetic chessboard.
- Encrypted calling-card string.
- GhostTown discussions containing information about the cipher.

**Challenge-provided SHA-256:**

```text
583450106b366b832250ee0dc30d57a055b27772387801d6dc77db9515cd354b
```

---

## 2. Technical Knowledge & Concepts

### 2.1 Chess Notation

Chess games are commonly recorded using algebraic notation.

For example:

| Notation | Meaning |
|---|---|
| `c4` | Pawn moves to c4 |
| `Nc3` | Knight moves to c3 |
| `Bg2` | Bishop moves to g2 |
| `Nf3` | Knight moves to f3 |
| `Bxe4` | Bishop captures a piece on e4 |
| `O-O` | Kingside castling |

Each square on a chessboard is identified by a file (`a`–`h`) and rank (`1`–`8`).

The challenge uses these coordinates as indexes into an alphabetic board.

### 2.2 Portable Game Notation (PGN)

PGN is a standard text format for recording chess games.

A PGN contains information such as the players, event, date, result, and complete move sequence.

Using a PGN parser makes it possible to reconstruct the chessboard after every move and determine exactly where each piece originated and landed.

### 2.3 Open-Source Intelligence (OSINT)

OSINT involves collecting and analyzing publicly available information.

In this challenge, a publicly recorded chess game serves as external key material.

The game itself is not secret. The difficulty lies in identifying the correct game and understanding how its moves relate to the cipher.

### 2.4 Vigenère Cipher

The Vigenère cipher is a polyalphabetic substitution cipher that uses a sequence of letters to determine character shifts.

Each key letter corresponds to a numerical value from 0 to 25.

Using zero-based alphabet indexing:

\[
P_i=(C_i-K_i)\bmod 26
\]

Where:

- \(P_i\) is the plaintext letter.
- \(C_i\) is the ciphertext letter.
- \(K_i\) is the corresponding key letter.

For this challenge, alphabetic characters are decrypted using the key while retaining their original case.

Numbers and underscores remain unchanged and do not consume key characters.

---

## 3. My Independent Investigation

### Investigating GhostTown

I located a GhostTown discussion describing a homemade encryption scheme inspired by Magnus Carlsen.

The author explained that:

- A chessboard had been mapped to alphabetic characters.
- Repeated letters were intentionally included.
- A Magnus Carlsen game from 2026 supplied key material.
- The game featured the English Opening.
- Only Magnus's moves were relevant.
- The relationship between the game and chessboard was essential to decoding the message.

These clues suggested that multiple pieces of public information needed to be combined.

### Creating an Investigation Workspace

I organized the investigation under:

```text
~/CTF-Evidence/Cryptography/Chess-Key/
```

My workspace included:

```text
Chess-Key/
├── .venv/
├── PGN/
├── find_carlsen.py
├── extract_moves.py
├── analyze_cipher.py
├── test_key_hypothesis.py
├── carlsen_candidates.csv
└── keymer_move_mapping.txt
```

The scripts supported game discovery, chess-coordinate extraction, ciphertext analysis, and exploratory key calculations.

The evidence workspace was kept separate from the public GitHub write-up.

### Searching Public Chess Game Records

I downloaded Lichess broadcast PGN archives and used Python to identify Magnus Carlsen games played during 2026.

The investigation used the `python-chess` and `zstandard` libraries.

Installation:

```bash
python3 -m venv .venv
source .venv/bin/activate

python -m pip install python-chess zstandard
```

The game discovery script:

```bash
python find_carlsen.py
```

It filtered game records using:

- Player names
- Date
- ECO opening classification
- Opening description
- Magnus Carlsen's color

One initial problem was that player names appeared as `Carlsen, Magnus`, whereas the script originally searched only for `Magnus Carlsen`.

The correction normalized the name before comparing it:

```python
def is_carlsen(name):
    normalized = name.casefold().replace(",", " ")
    parts = normalized.split()
    return "carlsen" in parts and "magnus" in parts
```

This allowed the script to detect games using either naming convention.

### Candidate Game Discovery

The following broadcast archives were scanned:

| Month | Games Scanned | English Opening Candidates |
|---|---:|---:|
| May 2026 | 37,447 | 1 |
| June 2026 | 36,996 | 2 |
| July 2026 | 40,038 | 0 |
| August 2026 | 48,940 | 0 |
| **Total** | **163,421** | **3** |

The three matching candidates were:

| Date | Opponent | Magnus's Color |
|---|---|---|
| May 24, 2026 (PGN date) | Vincent Keymer | White |
| June 11, 2026 | Manuel Peña Gómez | Black |
| June 11, 2026 | Xu Xiangyu | Black |

The May game against Vincent Keymer appeared particularly promising because Magnus played White and opened with `1.c4`.

This candidate was investigated further.

**Important:** These archives were not a complete record of Magnus Carlsen's online games. The correct game was ultimately identified from the external write-up rather than from this local candidate list.

### Extracting Chess Coordinates

I created `extract_moves.py` to parse the Carlsen–Keymer game and extract Magnus's starting and destination squares.

Example:

```text
1. c4       c2 → c4
2. Nc3      b1 → c3
3. Nf3      g1 → f3
4. dxc3     d2 → c3
5. e4       e2 → e4
```

The parser correctly reconstructed moves from the game state, including captures.

It extracted 85 Magnus moves.

The output was saved using:

```bash
python extract_moves.py > keymer_move_mapping.txt
```

The resulting evidence file contained 87 lines: a game identification header, a blank line, and 85 move records.

### Reconstructing the Alphabetic Chessboard

I manually transcribed the original 8×8 board.

Reading from rank 8 down to rank 1:

```text
8 | A I W Y A E I O
7 | J B K B F J P U
6 | X L C M K Q V A
5 | Z C N D O W B E
4 | D G L P E Q F I
3 | H M R X R F S K
2 | N S Y C G T G U
1 | T Z D H J L V H
    ---------------
    a b c d e f g h
```

The transcription was added to Python:

```python
ROWS = [
    "AIWYAEIO",
    "JBKBFJPU",
    "XLCMKQVA",
    "ZCNDOWBE",
    "DGLPEQFI",
    "HMRXRFSK",
    "NSYCGTGU",
    "TZDHJLVH",
]

FILES = "abcdefgh"

def square_letter(square):
    file_index = FILES.index(square[0])
    row_index = 8 - int(square[1])
    return ROWS[row_index][file_index]
```

This made it possible to retrieve letters automatically for any move coordinate.

Example output from the Keymer game:

```text
1. c4       c2 (Y) → c4 (L)
2. Nc3      b1 (Z) → c3 (R)
3. Nf3      g1 (V) → f3 (F)
4. dxc3     d2 (C) → c3 (R)
5. e4       e2 (G) → e4 (E)
```

The chessboard transcription was later consistent with the board shown in the referenced solution.

### Ciphertext Analysis and Unsuccessful Hypothesis

I analyzed the calling card:

```text
nZ3ipU8_yy3p5_hvyKi0
```

Results:

| Property | Result |
|---|---|
| Total length | 20 characters |
| Alphabetic characters | 13 |
| Digits | 5 |
| Underscores | 2 |
| Segment lengths | 7, 5, 6 |

One exploratory hypothesis was that each move supplied a shift calculated from the difference between its origin and destination letters.

For the first five moves in the Keymer game:

| Move | Letters | Difference |
|---|---|---:|
| c4 | Y → L | −13 |
| Nc3 | Z → R | −8 |
| Nf3 | V → F | −16 |
| dxc3 | C → R | +15 |
| e4 | G → E | −2 |

I tested those values against the first five ciphertext characters using a hypothetical 62-character alphabet.

The result was:

```text
A7j3r
```

This did not provide convincing evidence that the method was correct.

I had not identified the intended extraction rule or the correct game.

**This is where my independent investigation stopped. The remaining solution is credited to TheMusk3teers.**

---

## 4. Published Solution — TheMusk3teers

> **Source Attribution**
>
> The following explanation is adapted and independently worded from the public [Chess Key write-up by TheMusk3teers](https://github.com/TheMusk3teers/TheMusk3teers.github.io/blob/68b72eeed6a38d019cc18dee32be736ace8959ec/docs/ctfs/deadface-2026/cryptography/chess-key.md).
>
> Their team identified the correct chess game, established the destination-square extraction method, and recovered the flag.
>
> This section explains their published method; it does not represent an independently completed competition solve.

### Identify the Correct Chess Game

TheMusk3teers identified a Magnus Carlsen game played against Aleksandar Indjić on January 13, 2026.

**Game details:**

| Field | Value |
|---|---|
| Player | Magnus Carlsen |
| Opponent | Aleksandar Indjić |
| Opponent's Online Username | Beca95 |
| Date | January 13, 2026 |
| Event | Titled Tuesday |
| Magnus's Color | White |
| Opening | English Opening |
| Opening Move | `1. c4` |

[View the referenced Chess.com game](https://www.chess.com/game/live/162638222425)

The relevant opening moves were:

```text
1. c4 e5
2. g3 Nc6
3. Bg2 h5
4. Nf3 e4
5. Nh4 g5
6. Nf5 h4
7. Nc3 Nf6
8. d3 d6
9. g4 Nxg4
10. Bxe4 Nh6
11. Nxh6 Bxh6
12. Rg1 Ne5
13. d4
```

These moves were the key information missing from my own investigation.

### Derive the Vigenère Key

TheMusk3teers' method uses the **destination square of each Magnus move**, rather than the difference between its origin and destination.

Because the ciphertext contains 13 alphabetic characters, only the first 13 moves by Magnus are required.

Each destination coordinate is converted into a letter using the alphabetic chessboard.

| Move | Destination | Board Letter |
|---|---|---|
| `1. c4` | c4 | L |
| `2. g3` | g3 | S |
| `3. Bg2` | g2 | G |
| `4. Nf3` | f3 | F |
| `5. Nh4` | h4 | I |
| `6. Nf5` | f5 | W |
| `7. Nc3` | c3 | R |
| `8. d3` | d3 | X |
| `9. g4` | g4 | F |
| `10. Bxe4` | e4 | E |
| `11. Nxh6` | h6 | A |
| `12. Rg1` | g1 | V |
| `13. d4` | d4 | P |

The derived key is:

```text
LSGFIWRXFEAVP
```

This key is applied to the alphabetic characters of the ciphertext.

### Vigenère Decryption

The published solution applies conventional Vigenère subtraction independently to uppercase and lowercase letters.

Digits and underscores are preserved unchanged.

For example, using the first encrypted letter:

```text
Ciphertext: n
Key:        L

n = 13 (zero-based)
L = 11 (zero-based)

(13 - 11) mod 26 = 2

2 = c
```

The full character mapping is:

| Ciphertext | Key | Plaintext |
|---|---|---|
| n | L | c |
| Z | S | H |
| 3 | — | 3 |
| i | G | c |
| p | F | k |
| U | I | M |
| 8 | — | 8 |
| _ | — | _ |
| y | W | c |
| y | R | h |
| 3 | — | 3 |
| p | X | s |
| 5 | — | 5 |
| _ | — | _ |
| h | F | c |
| v | E | r |
| y | A | y |
| K | V | P |
| i | P | t |
| 0 | — | 0 |

The resulting plaintext is:

```text
cH3ckM8_ch3s5_cryPt0
```

This matches the chess and checkmate theme of the original calling card.

### Reproducing the Published Decryption

The following educational Python implementation demonstrates the Vigenère decryption step using the key reported by TheMusk3teers.

It is included as a reproduction example, not as a record of code used to independently solve the challenge.

```python
ciphertext = "nZ3ipU8_yy3p5_hvyKi0"
key = "LSGFIWRXFEAVP"

plaintext = []
key_index = 0

for char in ciphertext:
    if char.isascii() and char.isalpha():
        shift = ord(key[key_index]) - ord("A")

        if char.isupper():
            base = ord("A")
        else:
            base = ord("a")

        decoded = chr(
            (ord(char) - base - shift) % 26 + base
        )

        plaintext.append(decoded)
        key_index += 1
    else:
        plaintext.append(char)

result = "".join(plaintext)

print("Ciphertext:", ciphertext)
print("Key:", key)
print("Plaintext:", result)
```

**Expected result, according to the published solution:**

```text
Ciphertext: nZ3ipU8_yy3p5_hvyKi0
Key: LSGFIWRXFEAVP
Plaintext: cH3ckM8_ch3s5_cryPt0
```

The ciphertext has 13 letters, so the 13-character key is consumed exactly once.

---

## 5. Challenge Flag — External Reference

<details>
<summary><strong>🚩 Click to Reveal Referenced Flag — Spoiler Warning</strong></summary>

**Flag reported by TheMusk3teers:**

`deadface{cH3ckM8_ch3s5_cryPt0}`

**Source:** [TheMusk3teers — Chess Key](https://github.com/TheMusk3teers/TheMusk3teers.github.io/blob/68b72eeed6a38d019cc18dee32be736ace8959ec/docs/ctfs/deadface-2026/cryptography/chess-key.md)

**Personal competition status:** Unsolved.

**Personal platform verification:** Not performed.

**Points earned by me for this challenge:** 0.

</details>

---

## 6. References & Attribution

### Primary Solution Reference

**TheMusk3teers — Chess Key, DEADFACE CTF 2026**

Credited contributors:

- [H0ussamCl4p](https://github.com/H0ussamCl4p)
- [chaimae098](https://github.com/chaimae098)
- [fatimaezzahralag-ship-it](https://github.com/fatimaezzahralag-ship-it)
- [imanebouzbiba](https://github.com/imanebouzbiba)

[Read the original Chess Key write-up](https://github.com/TheMusk3teers/TheMusk3teers.github.io/blob/68b72eeed6a38d019cc18dee32be736ace8959ec/docs/ctfs/deadface-2026/cryptography/chess-key.md)

### Additional References

- [Chess.com — Carlsen vs. Indjić game](https://www.chess.com/game/live/162638222425)
- [GhostTown — Carlsen Gave Me an Idea](https://ghosttown.deadface.io/t/38/carlsen-gave-me-an-idea)
- [Lichess Open Database](https://database.lichess.org/)

---

[← Return to Cryptography Collection](../README.md)
