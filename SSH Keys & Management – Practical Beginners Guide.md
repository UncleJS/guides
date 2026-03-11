# SSH Keys & Management – Practical Beginner’s Guide

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)


SSH keys are the standard, secure way to authenticate to remote systems without passwords. Properly used, they are **more secure**, **automatable**, and **safer** than passwords.

This guide focuses on:

* How SSH keys work
* Creating keys correctly
* Deploying keys safely
* Managing multiple keys
* Security best practices
* Troubleshooting common failures

All examples assume a Linux environment (RHEL/RHEL-compatible).

---


## Table of Contents

- [SSH Keys \& Management – Practical Beginner’s Guide](#ssh-keys--management--practical-beginners-guide)
  - [Table of Contents](#table-of-contents)
  - [1. What SSH Keys Actually Are](#1-what-ssh-keys-actually-are)
  - [2. Default SSH Key Locations (Linux)](#2-default-ssh-key-locations-linux)
  - [3. Recommended Key Type (Modern Standard)](#3-recommended-key-type-modern-standard)
  - [4. Generating Your First Key (Correctly)](#4-generating-your-first-key-correctly)
  - [5. Understanding Generated Files](#5-understanding-generated-files)
  - [6. Installing Your Public Key on a Server](#6-installing-your-public-key-on-a-server)
    - [Option A – Easiest (If Password Login Works)](#option-a--easiest-if-password-login-works)
    - [Option B – Manual Method](#option-b--manual-method)
  - [7. Connecting Using a Specific Key](#7-connecting-using-a-specific-key)
  - [8. Managing Multiple Keys (Very Common)](#8-managing-multiple-keys-very-common)
  - [9. SSH Agent (Avoid Re-typing Passphrases)](#9-ssh-agent-avoid-re-typing-passphrases)
  - [10. Key Security Best Practices](#10-key-security-best-practices)
    - [✔ Always Use Passphrases](#-always-use-passphrases)
    - [✔ Never Reuse Keys Across Roles](#-never-reuse-keys-across-roles)
    - [✔ Protect Permissions](#-protect-permissions)
    - [✔ Disable Password Authentication (Servers)](#-disable-password-authentication-servers)
  - [11. Revoking / Removing Access](#11-revoking--removing-access)
  - [12. Verifying Which Key Is Used](#12-verifying-which-key-is-used)
  - [13. Common Problems \& Fixes](#13-common-problems--fixes)
    - [❌ Permission Denied (Publickey)](#-permission-denied-publickey)
    - [❌ Wrong Key Used](#-wrong-key-used)
    - [❌ Server Ignores Key](#-server-ignores-key)
    - [❌ SELinux Issues (RHEL Specific)](#-selinux-issues-rhel-specific)
  - [14. Backing Up SSH Keys (Important)](#14-backing-up-ssh-keys-important)
  - [15. When To Generate New Keys](#15-when-to-generate-new-keys)
  - [16. Good Operational Habits](#16-good-operational-habits)

---
## 1. What SSH Keys Actually Are

SSH authentication uses **asymmetric cryptography**:

| Component       | Purpose                                  | Location              |
| --------------- | ---------------------------------------- | --------------------- |
| **Private Key** | Secret credential used to prove identity | **Your machine only** |
| **Public Key**  | Non-secret identity verifier             | Remote server         |

**Key rule:**

> Never copy or share your private key.

---


[↑ Goto TOC](#table-of-contents)

## 2. Default SSH Key Locations (Linux)

Keys live in:

```
~/.ssh/
```

Common files:

| File              | Meaning                  |
| ----------------- | ------------------------ |
| `id_ed25519`      | Private key              |
| `id_ed25519.pub`  | Public key               |
| `authorized_keys` | Server-side allowed keys |
| `config`          | Client configuration     |
| `known_hosts`     | Host verification cache  |

Correct permissions are critical:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/*
```

---


[↑ Goto TOC](#table-of-contents)

## 3. Recommended Key Type (Modern Standard)

Use **ED25519** unless you have a legacy constraint.

Advantages:

* Smaller
* Faster
* Strong security
* Widely supported

Avoid generating new RSA keys unless required.

---


[↑ Goto TOC](#table-of-contents)

## 4. Generating Your First Key (Correctly)

Basic command:

```bash
ssh-keygen -t ed25519 -C "your_email_or_label"
```

Better practice with explicit filename:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_work -C "work-laptop"
```

You will be prompted for:

* **Passphrase** → Always use one (important)

Why passphrases matter:

* Protects stolen key files
* Prevents silent misuse

---


[↑ Goto TOC](#table-of-contents)

## 5. Understanding Generated Files

Example:

```
id_ed25519_work        ← PRIVATE KEY (secret)
id_ed25519_work.pub    ← PUBLIC KEY (safe)
```

View public key:

```bash
cat ~/.ssh/id_ed25519_work.pub
```

Looks like:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIB... work-laptop
```

---


[↑ Goto TOC](#table-of-contents)

## 6. Installing Your Public Key on a Server


[↑ Goto TOC](#table-of-contents)

### Option A – Easiest (If Password Login Works)

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_work user@server
```

---


[↑ Goto TOC](#table-of-contents)

### Option B – Manual Method

Append your public key to:

```
~/.ssh/authorized_keys
```

On server:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
vi ~/.ssh/authorized_keys
```

Paste key → Save → Fix permissions:

```bash
chmod 600 ~/.ssh/authorized_keys
```

---


[↑ Goto TOC](#table-of-contents)

## 7. Connecting Using a Specific Key

```bash
ssh -i ~/.ssh/id_ed25519_work user@server
```

Without specifying, SSH tries defaults.

---


[↑ Goto TOC](#table-of-contents)

## 8. Managing Multiple Keys (Very Common)

Use SSH config file:

```bash
vi ~/.ssh/config
```

Example:

```ini
Host work-server
    HostName 192.168.10.50
    User jaco
    IdentityFile ~/.ssh/id_ed25519_work

Host github
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github
```

Now connect cleanly:

```bash
ssh work-server
ssh github
```

---


[↑ Goto TOC](#table-of-contents)

## 9. SSH Agent (Avoid Re-typing Passphrases)

Start agent:

```bash
eval "$(ssh-agent -s)"
```

Add key:

```bash
ssh-add ~/.ssh/id_ed25519_work
```

Check loaded keys:

```bash
ssh-add -l
```

**Gotcha:**
Agent resets on reboot unless integrated into session manager.

---


[↑ Goto TOC](#table-of-contents)

## 10. Key Security Best Practices


[↑ Goto TOC](#table-of-contents)

### ✔ Always Use Passphrases

Unprotected keys = high risk.

---


[↑ Goto TOC](#table-of-contents)

### ✔ Never Reuse Keys Across Roles

Bad:

* Same key for GitHub + servers + clients

Good:

* Separate keys per environment

Example naming:

```
id_ed25519_personal
id_ed25519_work
id_ed25519_servers
```

---


[↑ Goto TOC](#table-of-contents)

### ✔ Protect Permissions

SSH refuses insecure keys.

Correct:

```bash
chmod 600 ~/.ssh/id_ed25519*
```

---


[↑ Goto TOC](#table-of-contents)

### ✔ Disable Password Authentication (Servers)

In `/etc/ssh/sshd_config`:

```ini
PasswordAuthentication no
PubkeyAuthentication yes
```

Reload:

```bash
sudo systemctl reload sshd
```

**Warning:**
Ensure keys work before disabling passwords.

---


[↑ Goto TOC](#table-of-contents)

## 11. Revoking / Removing Access

Simply delete the key line from:

```
~/.ssh/authorized_keys
```

No restart required.

---


[↑ Goto TOC](#table-of-contents)

## 12. Verifying Which Key Is Used

Verbose mode:

```bash
ssh -v user@server
```

Look for:

```
Offering public key: ~/.ssh/id_ed25519_work
```

---


[↑ Goto TOC](#table-of-contents)

## 13. Common Problems & Fixes

---


[↑ Goto TOC](#table-of-contents)

### ❌ Permission Denied (Publickey)

Check:

```bash
ls -ld ~/.ssh
ls -l ~/.ssh
```

Must be:

* `.ssh` → 700
* `authorized_keys` → 600

---


[↑ Goto TOC](#table-of-contents)

### ❌ Wrong Key Used

Force key:

```bash
ssh -i ~/.ssh/id_ed25519_work user@server
```

Or fix config.

---


[↑ Goto TOC](#table-of-contents)

### ❌ Server Ignores Key

On server:

```bash
sudo grep PubkeyAuthentication /etc/ssh/sshd_config
```

Must be:

```
PubkeyAuthentication yes
```

---


[↑ Goto TOC](#table-of-contents)

### ❌ SELinux Issues (RHEL Specific)

Restore contexts:

```bash
restorecon -Rv ~/.ssh
```

Very common on RHEL systems.

---


[↑ Goto TOC](#table-of-contents)

## 14. Backing Up SSH Keys (Important)

You **must** keep secure backups of private keys.

Safe approach:

* Encrypted storage
* Offline backup
* Never cloud-sync unencrypted keys

To copy keys:

```bash
cp -a ~/.ssh /secure/location/
```

---


[↑ Goto TOC](#table-of-contents)

## 15. When To Generate New Keys

Generate new keys if:

* Device lost/stolen
* Private key exposed
* Role separation needed
* Periodic rotation policy

---


[↑ Goto TOC](#table-of-contents)

## 16. Good Operational Habits

✔ Use descriptive filenames </br>
✔ Separate keys by trust boundary </br>
✔ Keep config clean </br>
✔ Remove unused keys </br>
✔ Use passphrases always </br>



[↑ Goto TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
