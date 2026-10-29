
# Looking Back — DEADFACE CTF 2026

## Challenge Information

| Field | Details |
|---|---|
| Competition | DEADFACE CTF 2026 |
| Category | Hostbusters |
| Challenge | Looking Back |
| Challenge Author | @syyntax |
| Points | 30 |
| Competition Status | ✅ Personal Solve — During CTF |
| Points Earned | 30 |
| Environment | Linux Web Shell |
| Initial Account | `svc-deploy` |
| Target Account | `deephax` |
| Target Host | `charon` |
| Techniques | Shell History Analysis, Credential Discovery, Linux Account Switching, Host Enumeration |

---

## 1. Challenge Description & Objective

### Scenario

Whichever member of DEADFACE used the svc-deploy account last was sloppy. There are many DEADFACE members who use this machine.

Access one of their accounts and read the flag in their home directory.

NOTE: Heads up: charon is disposable. Every web-shell session is a fresh copy of the box, so grab each flag as you find it.

### Objective

Investigate the available evidence on Charon, identify credentials or other useful information left behind by a previous user, and gain access to another DEADFACE account.

Once authenticated, locate and retrieve the flag from the second user's home directory.

**Required flag format:**

`deadface{flag-text}`

### Challenge Environment

The challenge provided access through the **Access Hostbusters** web shell.

The initial environment was:

```text
Hostname: charon
Username: svc-deploy
Home:     /home/svc-deploy
```

The Hostbusters environment was disposable, meaning new sessions could start from a clean instance of the target.

---

## 2. Technical Concepts

### 2.1 Linux Shell History

Bash can store previously executed commands in a file named:

```text
~/.bash_history
```

This file can provide useful information during a forensic investigation.

Depending on the user's activity, shell history may reveal:

- Previously executed commands
- Accessed files and directories
- Administrative activity
- User account management
- Application configuration
- Accidentally exposed credentials

Because command history may contain sensitive information, it can become a security risk if improperly managed.

### 2.2 Credential Exposure

Credentials should not be placed directly into shell commands when avoidable.

Commands containing plaintext passwords may remain visible through:

- Shell history files
- Terminal logs
- Process information, depending on how commands are executed
- Administrative monitoring systems
- Captured command output

In this challenge, a password was exposed through a previous administrative command.

### 2.3 The `chpasswd` Command

Linux provides the `chpasswd` utility for updating account passwords.

The utility accepts input in this format:

```text
username:password
```

For example:

```bash
echo "exampleuser:examplepassword" | sudo chpasswd
```

This passes a username and password to `chpasswd` through standard input.

Although useful for automated account management, embedding credentials directly in a shell command can expose sensitive information through command history.

### 2.4 The `su` Command

The `su` command allows a user to switch to another account when the required authentication and system permissions permit it.

Syntax:

```bash
su username
```

Alternatively:

```bash
su - username
```

The second form starts a login shell and initializes the target user's login environment.

Both forms generally require the target account's password when invoked by an ordinary user.

### 2.5 Linux File Permissions

Linux controls access to files and directories using ownership and permission bits.

For example:

```text
-rw-------
```

This indicates that the owner can read and write the file, while other users have no access.

The `ls -lah` command displays file permissions, ownership, sizes, and timestamps, including hidden files.

---

## 3. Investigation Methodology

### Access the Charon Web Shell

I accessed the Hostbusters environment through the challenge-provided web shell.

The initial session ran under the `svc-deploy` account.

```text
charon:~$
```

This account had been used in the previous Hostbusters challenge, Foothold on Charon.

The first objective was to identify files and artifacts available from its home directory.

### Enumerate the Home Directory

I executed:

```bash
ls -lah
```

Output:

```text
total 16K
drwxr-x---    2 svc-depl svc-depl    4.0K Oct 18 01:14 .
drwxr-xr-x    1 root     root        4.0K Oct 18 01:14 ..
-rw-------    1 svc-depl svc-depl     196 Oct 18 01:14 .bash_history
-rw-r-----    1 svc-depl svc-depl      33 Oct 18 01:14 flag.txt
```

Two files were identified:

| File | Permissions | Purpose |
|---|---|---|
| `.bash_history` | `-rw-------` | Previous shell commands |
| `flag.txt` | `-rw-r-----` | Flag from the initial foothold challenge |

The `.bash_history` file was particularly relevant because the challenge title, **Looking Back**, suggested investigating previous activity.

### Examine Shell History

I opened the history file:

```bash
cat .bash_history
```

Output:

```bash
cd /opt/deploy
cat README.txt
systemctl status charon-collector.service || true
# ticket OPS-4471: rotate member pw after onboarding
echo "deephax:Sk4t3rB0y_S1nce_2011" | sudo chpasswd
history -c
```

This output contained several useful clues.

#### Finding 1 — Deployment Directory

```bash
cd /opt/deploy
```

This indicated that the previous operator had accessed a deployment-related directory.

#### Finding 2 — Service Investigation

```bash
systemctl status charon-collector.service || true
```

The operator had inspected the status of a service named `charon-collector.service`.

The `|| true` portion prevents a nonzero exit status from causing the overall command to fail.

#### Finding 3 — Password Rotation Ticket

```text
# ticket OPS-4471: rotate member pw after onboarding
```

This comment suggested that the operator was performing a password rotation related to member onboarding.

#### Finding 4 — Exposed Credentials

The most important command was:

```bash
echo "deephax:Sk4t3rB0y_S1nce_2011" | sudo chpasswd
```

The `chpasswd` utility expects input formatted as:

```text
username:password
```

Therefore, the command exposed credentials associated with the `deephax` account.

| Credential Field | Recovered Value |
|---|---|
| Username | `deephax` |
| Password | `Sk4t3rB0y_S1nce_2011` |
| Source | `.bash_history` |
| Associated Action | Password rotation |

The successful authentication later confirmed that the recovered password was valid.

### Examine the History-Clearing Attempt

The final command was:

```bash
history -c
```

This command clears the current shell's in-memory history list.

However, the command was still visible in the `.bash_history` file recovered during the investigation.

This illustrated an important distinction between clearing an active shell's history and ensuring that sensitive entries are absent from a persistent history file.

Regardless of the operator's intent, the sensitive password remained recoverable.

### Identify the Target Account

The credential pair revealed the username:

```text
deephax
```

Initially, I considered whether the text following the colon might represent another username.

Reviewing the expected `chpasswd` input format clarified that `deephax` was the account name and the remaining value was its password.

This interpretation was consistent with the preceding password-rotation comment.

### Switch to the Target Account

Using the credentials discovered in shell history, I attempted to switch accounts with `su`.

The account-switching command was:

```bash
su - deephax
```

An initial authentication attempt was unsuccessful.

After confirming the username/password distinction and retrying authentication, I successfully accessed the `deephax` account.

The password was entered at the interactive password prompt rather than embedded in another shell command.

This demonstrated that the credentials exposed in `.bash_history` could be used to access another local account.

### Recover the Flag

After successfully switching to the target account, I searched the accessible home-directory contents and recovered the challenge flag.

The recovered value was:

```text
deadface{sh3ll_h1st0ry_t3lls_4ll}
```

The flag matched the expected DEADFACE format.

The exact flag-file path and final retrieval command were not preserved in the available terminal transcript, so they are not reconstructed as verified commands here.

### Confirm the Solution

The recovered flag was confirmed as correct.

```text
deadface{sh3ll_h1st0ry_t3lls_4ll}
```

The challenge was personally solved during DEADFACE CTF 2026.

No external walkthrough or published solution was required.

---

## 4. Challenge Flag

<details>
<summary><strong>🚩 Click to Reveal Flag — Spoiler Warning</strong></summary>

**Recovered Flag:**

`deadface{sh3ll_h1st0ry_t3lls_4ll}`

**Competition Status:** ✅ Personally Solved During CTF

**Points Earned:** 30

**External Write-up Used:** None

</details>

---

## 5. References

- [Linux man-pages — su(1)](https://man7.org/linux/man-pages/man1/su.1.html)
- [Linux man-pages — chpasswd(8)](https://man7.org/linux/man-pages/man8/chpasswd.8.html)
- [GNU Bash Reference Manual — History](https://www.gnu.org/software/bash/manual/html_node/Bash-History-Facilities.html)
- [MITRE ATT&CK — Credentials in Files](https://attack.mitre.org/techniques/T1552/001/)
- [MITRE ATT&CK — Valid Accounts](https://attack.mitre.org/techniques/T1078/)

---

[← Return to Hostbusters Collection](../README.md)
