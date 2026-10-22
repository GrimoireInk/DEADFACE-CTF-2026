# Back to Basics — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Challenge Set | Aurora Patient Zero |
| Challenge | Back to Basics |
| Category | Mobile Application Security |
| Points | 15 |
| Author | @G2Gh0st |
| Target | Aurora Patient Portal (iOS) |
| Status | Solved |

---

## 1. Challenge Description & Objective

### Scenario

For this task, you need to continue your examination of the Aurora Patient Portal application.

Now we know how DEADFACE is able to collect all this information, but now we would like to deploy some network monitoring to ensure we know everything they are doing. There is a hidden diagnostic page built into the app. Using ALL the possible data that you were able to collect so far, see if you have search for the name of the gateway server the app is using to communicate back to Aurora's servers.

### Objective

Identify the name of the application's gateway server.

**Required flag format:** `deadface{gateway-name}`

No application execution or network capture was required for the successful solution.

---

## 2. Technical Knowledge & Concepts

### 2.1 Static Application Analysis

Static analysis involves inspecting application files without executing the program.

For this challenge, the Aurora Patient Portal's compiled iOS executable was examined for readable strings associated with diagnostic functionality.

This technique can reveal embedded configuration information, application labels, debugging messages, and other text present in the executable.

### 2.2 What Is the `strings` Command?

The `strings` utility extracts sequences of printable characters from binary files.

Example:

```bash
strings -a application_binary
```

The `-a` option scans the entire file rather than restricting the search to selected sections.

While `strings` can expose useful information, its output does not necessarily establish how the application uses each recovered value.

### 2.3 What Is `grep`?

`grep` is a command-line utility for searching text using patterns.

For this investigation, the following options were used:

| Option | Purpose |
|---|---|
| `-i` | Case-insensitive searching |
| `-E` | Extended regular expressions |
| `-B 20` | Show 20 lines before a match |
| `-A 30` | Show 30 lines after a match |

Piping the output of `strings` into `grep` allowed diagnostic-related information to be located quickly.

### 2.4 What Is a Gateway Server?

A gateway server acts as an intermediary between systems or networks.

In a mobile application architecture, a gateway may handle communication between an application and backend services.

A gateway hostname or identifier can provide useful information about an organization's infrastructure, particularly when it reveals internal naming conventions or environment details.

The presence of a hostname in a binary does not by itself establish that the server is currently reachable or operational.

---

## 3. Investigation Methodology

### Phase 1 — Evidence Preparation

#### Objective

Prepare a dedicated workspace and obtain the Aurora Patient Portal executable from the previously extracted iOS application.

#### Step 1: Navigate to the Investigation Directory

```bash
cd ~/CTF-Evidence/Aurora-Patient-Zero/Back-to-Basics
```

#### Step 2: Copy the Application Executable

```bash
cp "../Down-the-Rabbit-Hole/extracted-ipa/Payload/aurora-patient-portal.app/Aurora Health Mobile" .
```

This copied the executable into the Back-to-Basics investigation workspace without modifying the earlier evidence.

#### Step 3: Verify the File

```bash
ls -lh
```

**Evidence examined:** `Aurora Health Mobile`

---

### Phase 2 — Discover Diagnostic References

#### Objective

Determine whether the application executable contains references to the hidden diagnostic screen.

#### Step 1: Search for Diagnostic and Network Strings

```bash
strings -a "Aurora Health Mobile" | grep -iE 'diagnostic|gateway|server|network|debug|healthcheck' | head -n 60
```

#### Actual Output

```text
DiagnosticScreen
DiagnosticRow
DiagnosticItem
_isShowingDiagnosticScreen
Last Gateway Check
Aurora Diagnostics
```

#### Analysis

Several references suggested the presence of diagnostic-related functionality:

- `DiagnosticScreen` indicated a diagnostic screen component.
- `DiagnosticRow` and `DiagnosticItem` suggested structured diagnostic information.
- `_isShowingDiagnosticScreen` suggested a state variable related to visibility.
- `Last Gateway Check` indicated gateway-related diagnostic information.
- `Aurora Diagnostics` appeared to be a diagnostic interface title.

These findings aligned with the challenge description.

---

### Phase 3 — Recover the Gateway Identifier

#### Objective

Examine additional strings surrounding the gateway diagnostic label.

#### Step 1: Expand the Search Context

```bash
strings -a "Aurora Health Mobile" | grep -i -B 20 -A 30 "Last Gateway Check"
```

#### Relevant Output

```text
aurora_chest_xray.jpg
heart.text.square.fill
Water target met
Blood Test review
Mental health therapy
AuroraPatientBootstrap
Linked study asset:
plus.magnifyingglass
Schedule a review session with your doctor
Demo chart for Patient:
All information is fictional and not suitable for actual healthcare decisions.
CTF patient companion
gw-westlake-int-03
Patient Cache Realm
patient-summary-cache-v2
0 pending records
Imaging Link Mode
local bundled asset
Medication Cache
offline lookup enabled
Last Gateway Check
2026-05-30 09:42 UTC
Aurora Diagnostics
Internal clinic sync status
```

#### Analysis

The expanded strings output revealed:

**Gateway identifier:** `gw-westlake-int-03`

The value appeared near other application diagnostic and configuration strings.

It was particularly relevant because:

1. The prefix `gw` suggested a gateway naming convention.
2. `westlake` matched the clinic value recovered during the Info Leaks investigation.
3. The executable contained related diagnostic labels, including `Last Gateway Check` and `Aurora Diagnostics`.

Although string proximity does not prove a runtime relationship, the challenge's accepted flag confirmed that this was the intended gateway identifier.

---

### Phase 4 — Flag Construction & Validation

The challenge required:

`deadface{gateway-name}`

The recovered gateway name was inserted into the required format.

**Validation result:** The DEADFACE CTF platform accepted the submitted flag.

**Challenge status:** Solved.

---

## 4. Findings & Evidence

| Evidence | Finding | Significance |
|---|---|---|
| `Aurora Health Mobile` | iOS application executable | Source of static analysis |
| `DiagnosticScreen` | Diagnostic component reference | Supports existence of diagnostic functionality |
| `_isShowingDiagnosticScreen` | Diagnostic visibility state reference | Suggests the screen may be conditionally displayed |
| `Last Gateway Check` | Gateway-related label | Supports investigation direction |
| `gw-westlake-int-03` | Gateway identifier | Required challenge answer |
| Previous Info Leaks findings | Clinic: Westlake | Corroborates the identifier's naming convention |
| DEADFACE flag submission | Accepted | Confirms successful solution |

### Investigation Conclusion

The gateway identifier was recovered directly from printable strings embedded in the Aurora Patient Portal executable.

The investigation demonstrated how application binaries may reveal diagnostic information even when the corresponding functionality is hidden from the normal user interface.

---

## 7. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{gw-westlake-int-03}`

**Status:** Accepted — Challenge Solved

</details>
