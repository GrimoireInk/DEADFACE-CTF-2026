
# Foothold on Charon — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Hostbusters |
| Challenge | Foothold on Charon |
| Challenge Author | @syyntax |
| Points Available | 10 |
| Investigation Status | Solved |
| Environment | Linux Web Shell |
| Initial User | `svc-deploy` |
| Target Hostname | `charon` |
| Techniques | Linux Enumeration, File Permissions Analysis, Initial Foothold Verification |

---

## 1. Challenge Description & Objective

### Scenario

DEADFACE runs an internal ops/staging box they call charon - it's where the crew coordinates their influence operation. We got you a foothold on it.

Open the web-shell by clicking the link below. You'll land at a shell as the svc-deploy service account. Get your bearings and find the flag that proves your foothold.

NOTE: Heads up: charon is disposable. Every web-shell session is a fresh copy of the box, so grab each flag as you find it.

### Objective

Identify the current operating environment, inspect the accessible files, and retrieve the flag that demonstrates the initial foothold.

**Required flag format:**

`deadface{flag-text}`

### Challenge Access

The challenge provided an **Access Hostbusters** link that opened a web-based Linux shell.

The provided session began with access to the target system as `svc-deploy`.

### Environment Characteristics

The challenge stated that Charon was disposable.

Every new web-shell session creates a fresh instance of the target environment.

Consequently, files or changes from one session should not be assumed to persist into another.

This makes it important to document findings and recover challenge flags during each session.

---

## 2. Technical Concepts

### 2.1 Initial Foothold

An initial foothold is a level of access obtained on a target system.

Depending on the circumstances, a foothold may provide limited user permissions or access to particular resources.

In this challenge, the initial foothold was already established.

The task was to inspect the existing access and prove control of the assigned service account.

### 2.2 Linux Service Accounts

Linux service accounts are frequently used to run applications, scheduled tasks, or deployment processes.

Such accounts may have permissions tailored to the specific services they support.

The challenge provided access through:

```text
svc-deploy
```

The account name suggests a deployment-related purpose, although the challenge did not require proving which services it normally manages.

### 2.3 Linux User Identification

Several commands help establish the current execution context.

| Command | Purpose |
|---|---|
| `whoami` | Displays the current username |
| `hostname` | Displays the system hostname |
| `pwd` | Shows the current working directory |
| `id` | Displays user and group identifiers |

These commands are useful during initial Linux enumeration.

### 2.4 Linux File Permissions

Linux file permissions control which users and groups can read, write, or execute files.

Permissions are commonly represented using three groups:

```text
Owner | Group | Others
```

For example:

```text
-rw-r-----
```

This means:

- The owner can read and write.
- Members of the file's group can read.
- Other users have no access.

Understanding these permissions helps determine which files an account can access without escalating privileges.

### 2.5 Hidden Files

Linux filenames beginning with a period are generally hidden from ordinary directory listings.

For example:

```text
.bash_history
```

The `ls -a` option includes hidden files in directory output.

Hidden files may contain useful information during an investigation, but their presence alone does not indicate malicious activity.

---

## 3. Investigation Methodology

### Access the Web Shell

I opened the web shell through the challenge's **Access Hostbusters** link.

The shell displayed the following prompt:

```text
charon:~$
```

The prompt suggested that I was connected to Charon and located in the user's home directory.

I verified this information using basic Linux commands.

### Identify the Current User

I executed:

```bash
whoami
```

Output:

```text
svc-deploy
```

This confirmed that the web shell was running under the service account specified in the challenge description.

### Identify the Current Directory

I ran:

```bash
pwd
```

Output:

```text
/home/svc-deploy
```

The current working directory was the home directory of `svc-deploy`.

This was an appropriate starting location for inspecting files accessible to the account.

### Confirm the Hostname

I executed:

```bash
hostname
```

Output:

```text
charon
```

This confirmed that the shell was connected to the expected challenge host.

### Inspect User and Group Identifiers

I executed:

```bash
id
```

Output:

```text
uid=1001(svc-deploy) gid=1001(svc-deploy) groups=1001(svc-deploy),1001(svc-deploy)
```

The output established:

| Property | Value |
|---|---|
| Username | `svc-deploy` |
| UID | 1001 |
| Primary GID | 1001 |
| Primary Group | `svc-deploy` |
| Hostname | `charon` |

The session was running as a non-root user.

There was no need to attempt privilege escalation for the challenge objective.

### Enumerate the Home Directory

With the account and working directory confirmed, I listed the contents of the home directory:

```bash
ls -lah
```

The command produced:

```text
total 16K
drwxr-x---    2 svc-depl svc-depl    4.0K Oct 18 01:14 .
drwxr-xr-x    1 root     root        4.0K Oct 18 01:14 ..
-rw-------    1 svc-depl svc-depl     196 Oct 18 01:14 .bash_history
-rw-r-----    1 svc-depl svc-depl      33 Oct 18 01:14 flag.txt
```

Two files were identified:

| File | Permissions | Observation |
|---|---|---|
| `.bash_history` | `-rw-------` | Hidden shell history file |
| `flag.txt` | `-rw-r-----` | Readable flag file |

The presence of `flag.txt` provided a direct lead.

The file was owned by the current account, and its permissions allowed the owner to read it.

### Read the Flag File

I executed:

```bash
cat flag.txt
```

The shell returned:

```text
deadface{f00th0ld_p4st_th3_v31l}
```

The contents matched the required DEADFACE flag format.

The challenge flag was recovered successfully.

### Verify the Solution

The recovered flag was:

```text
deadface{f00th0ld_p4st_th3_v31l}
```

The flag was confirmed as correct.

No additional exploitation, credential discovery, file modification, or privilege escalation was required.

The investigation was completed using the permissions already granted to the `svc-deploy` account.

---

## 4. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Recovered Flag:**

`deadface{f00th0ld_p4st_th3_v31l}`

**Investigation Status:** Solved

**Solver:** Independent / Personal Solve

**Competition Timing:** Not Confirmed

**External Assistance:** None

</details>

---

## 5. References

### Technical Documentation

- [Linux man-pages Project](https://man7.org/linux/man-pages/)
- [GNU Coreutils Manual](https://www.gnu.org/software/coreutils/manual/)
- [MITRE ATT&CK — System Owner/User Discovery](https://attack.mitre.org/techniques/T1033/)
- [MITRE ATT&CK — File and Directory Discovery](https://attack.mitre.org/techniques/T1083/)

---

[← Return to Hostbusters Collection](../README.md)
