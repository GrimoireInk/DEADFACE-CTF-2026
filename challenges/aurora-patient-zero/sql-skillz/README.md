# SQL Skillz — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| CTF | DEADFACE CTF 2026 |
| Challenge Set | Aurora Patient Zero |
| Challenge | SQL Skillz |
| Category | Mobile Security / Database Analysis |
| Points | 50 |
| Author | @G2Gh0st |
| Application | Aurora Patient Portal |
| Platform | iOS |
| Evidence | `aurora_meds.sqlite` |
| Status | Solved |

---

## 1. Challenge Description & Objective

### Scenario

For this task, you need to continue your examination of the Aurora Patient Portal application.

After completing Info Leaks and Down the Rabbit Hole, you have collected the patient data that is needed to query the application database file. Use the data collected to determine what medications the current patient is taking other than Ibuprofen.

### Objective

Identify the medication associated with the previously recovered Patient ID, excluding Ibuprofen.

**Required flag format:**

`deadface{Name}`

---

## 2. Technical Knowledge & Concepts

### 2.1 What Is SQLite?

SQLite is a lightweight, embedded relational database management system.

Unlike PostgreSQL or MySQL, SQLite does not require a separate database server. Its database is typically stored in a local file.

SQLite is commonly used in mobile applications to store structured information.

**Relevance to this challenge:**

The Aurora Patient Portal application contains a database named `aurora_meds.sqlite` with medication records.

### 2.2 What Is a Database Schema?

A database schema describes the organization of a database, including tables, columns, data types, and constraints.

This defines a table named `patients` containing a unique identifier and a required name.

**Relevance to this challenge:**

Examining the schema identified a `medications` table with a `pid` column and two medication-name columns.

### 2.3 What Is a Primary Key?

A primary key uniquely identifies each record within a table.

In this challenge, the database defines:

```sql
pid INTEGER PRIMARY KEY
```

This allows a specific medication record to be retrieved using the patient's unique ID.

### 2.4 What Are SQL SELECT and WHERE?

`SELECT` retrieves information from database tables.

`WHERE` filters records according to a condition.

Example:

```sql
SELECT *
FROM medications
WHERE pid = 123456;
```

This retrieves the record associated with the specified ID.

**Relevance to this challenge:**

A `WHERE` clause allowed the medication record to be isolated using the Patient ID recovered during Down the Rabbit Hole.

### 2.5 What Is Read-Only Database Analysis?

Read-only analysis allows investigators to inspect a database without intentionally modifying its contents.

SQLite's command-line utility provides a `-readonly` option.

Example:

```bash
sqlite3 -readonly database.sqlite ".tables"
```

**Security relevance:**

Preserving original evidence is important in forensic investigations. Using read-only access reduces the risk of accidentally changing records during examination.

---

## 3. Investigation Methodology

### Phase 1 — Evidence Preparation

Prepare a dedicated workspace and obtain the medication database from the previously extracted Aurora Patient Portal application.

#### Step 1: Create the Investigation Workspace

```bash
cd ~/CTF-Evidence/Aurora-Patient-Zero

mkdir -p SQL-Skillz

cd SQL-Skillz
```

**Command explanation:**

- `cd` changes directories.
- `mkdir` creates a directory.
- `-p` creates missing parent directories if necessary.

#### Step 2: Copy the SQLite Database

The database was previously identified within the extracted application bundle.

```bash
cp ../Down-the-Rabbit-Hole/extracted-ipa/Payload/aurora-patient-portal.app/aurora_meds.sqlite .
```

**Command explanation:**

- `cp` copies the database into the current investigation workspace.
- The final `.` represents the current directory.

This creates a working copy without modifying the database in the earlier investigation folder.

**Evidence file:** `aurora_meds.sqlite`

---

### Phase 2 — Database Structure Examination 

Identify the database's structure and determine how medication records are associated with patients.

#### Step 1: Examine the Medication Table Schema

```bash
sqlite3 -readonly aurora_meds.sqlite ".schema medications"
```

**Command explanation:**

- `sqlite3` opens the database.
- `-readonly` prevents database modification.
- `.schema medications` displays the SQL definition of the specified table.

**Actual output:**

    CREATE TABLE medications (
        pid INTEGER PRIMARY KEY,
        medication_1 TEXT NOT NULL,
        medication_1_dosage TEXT NOT NULL,
        medication_1_frequency TEXT NOT NULL,
        medication_2 TEXT NOT NULL,
        medication_2_dosage TEXT NOT NULL,
        medication_2_frequency TEXT NOT NULL

CREATE INDEX idx_medications_medication_1
ON medications(medication_1);

CREATE INDEX idx_medications_medication_2
ON medications(medication_2);

#### Step 2: Interpret the Schema

| Column | Type | Purpose |
|---|---|---|
| `pid` | INTEGER | Unique patient identifier |
| `medication_1` | TEXT | First medication |
| `medication_1_dosage` | TEXT | First medication dosage |
| `medication_1_frequency` | TEXT | First medication frequency |
| `medication_2` | TEXT | Second medication |
| `medication_2_dosage` | TEXT | Second medication dosage |
| `medication_2_frequency` | TEXT | Second medication frequency |

**Finding:**

The `medications` table stores two medication entries per patient record, including dosage and frequency information.

The `pid` column serves as the table's primary key.

Two indexes are also defined for searching medication names.

---

### Phase 3 — Patient Medication Investigation

Retrieve the medications associated with the Patient ID recovered in the previous challenge.

#### Step 1: Identify the Patient ID

The previous challenge, [Down the Rabbit Hole](../down-the-rabbit-hole/README.md), revealed the patient's identifier through metadata embedded in a JPEG image.

**Recovered Patient ID:** `847261`

This value was used to filter the medication table.

#### Step 2: Query the Patient Record

```bash
sqlite3 -readonly -header -column aurora_meds.sqlite \
  "SELECT * FROM medications WHERE pid = 847261;"
```

**Command explanation:**

- `-readonly` opens the database without permitting changes.
- `-header` displays column names.
- `-column` formats the output as aligned columns.
- `SELECT *` retrieves all columns.
- `FROM medications` identifies the table.
- `WHERE pid = 847261` filters the results by Patient ID.

**Finding:**

The matching patient record was used to identify the medication required by the challenge.

After excluding Ibuprofen, the medication was determined to be **Candesartan**.

The complete original query output was not preserved in the investigation notes, so no reconstructed terminal output is presented here.

#### Step 3: Construct and Validate the Flag

The challenge required:

`deadface{Name}`

Using the identified medication name, the flag was constructed and submitted.

**Validation result:** Accepted by the DEADFACE CTF platform.

---

## 4. Findings & Evidence

| Evidence | Finding | Significance |
|---|---|---|
| `aurora_meds.sqlite` | SQLite 3 database | Contains medication records |
| `medications` table | Seven defined columns | Establishes record structure |
| `pid` column | Primary key | Supports patient-specific lookup |
| Patient ID `847261` | Recovered previously | Links the investigations |
| Medication lookup | Candesartan | Answer to the challenge |
| Flag submission | Accepted | Confirms the answer |

### Investigation Conclusion

The medication information was recovered by querying a SQLite database bundled with the Aurora Patient Portal application.

The Patient ID obtained in Down the Rabbit Hole served as the lookup key, demonstrating how separate information exposures can be correlated to recover additional patient-related information.

---

## 5. Security Implications

### Exposure of Sensitive Medical Information

Medication records are sensitive healthcare information. If such records are included in a distributable application package, someone with access to that package may be able to examine them without authenticating to the application.

The challenge illustrates how a patient identifier exposed in one resource may be used to locate related medical data in another.

### Defensive Recommendations

1. Avoid packaging real patient medication records into publicly distributed application bundles.
2. Store sensitive patient data in systems protected by appropriate authentication and authorization.
3. Minimize the exposure of patient identifiers in application resources.
4. Implement access controls for retrieving patient-specific medical data.
5. Review application packages for sensitive embedded databases before release.

---

## 6. Lessons Learned

This challenge reinforced my understanding of:

- SQLite and embedded application databases
- Database schemas and column definitions
- Primary keys and record identification
- SQL `SELECT` and `WHERE` queries
- Read-only forensic database examination
- Correlating findings across multiple security challenges
- Identifying information disclosure risks involving medical records

---

## 7. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Confirmed Flag:**

`deadface{Candesartan}`

**Status:** Accepted — Challenge Solved

</details>
