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


[↑ Back to TOC](#table-of-contents)

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

Correct permissions are critical. Set them per file type, not with a blanket `chmod 600 ~/.ssh/*` (that would also hit public keys and any sub-directories, such as a `~/.ssh/sockets/` folder for multiplexing, which then become unusable):

| Path                                 | Mode  | Why                                     |
| ------------------------------------ | ----- | --------------------------------------- |
| `~/.ssh/`                            | `700` | Only you may list or enter it           |
| Private keys (`id_ed25519`, ...)     | `600` | Secret; SSH refuses keys others can read |
| `authorized_keys`                    | `600` | sshd ignores it if others can write it  |
| `config`                             | `600` | ssh refuses a config others can write   |
| Public keys (`*.pub`)                | `644` | Not secret; readable is fine            |
| `known_hosts`                        | `644` | Not secret (`600` also works)           |

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519 ~/.ssh/config ~/.ssh/authorized_keys
chmod 644 ~/.ssh/*.pub
```

Only run `chmod` on files that exist; skip any that are missing on your machine.

---


[↑ Back to TOC](#table-of-contents)

## 3. Recommended Key Type (Modern Standard)

Use **ED25519** unless you have a legacy constraint.

Advantages:

* Smaller
* Faster
* Strong security
* Widely supported

Avoid generating new RSA keys unless required. If you must use RSA (for example, for an old system that doesn't support Ed25519), use **at least 3072 bits**; 4096 is a common choice:

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_legacy -C "legacy-system"
```

Never create new RSA keys smaller than 3072 bits. RHEL/Fedora crypto-policies already reject RSA keys under 2048 bits.

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

## 6. Installing Your Public Key on a Server


[↑ Back to TOC](#table-of-contents)

### Option A – Easiest (If Password Login Works)

```bash
ssh-copy-id -i ~/.ssh/id_ed25519_work user@server
```

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

## 7. Connecting Using a Specific Key

```bash
ssh -i ~/.ssh/id_ed25519_work user@server
```

Without specifying, SSH tries defaults.

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

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

**GNOME desktops (Fedora Workstation, RHEL with GNOME):** an agent is usually already running for you, provided by gnome-keyring / `gcr-ssh-agent`. Check before starting another one:

```bash
echo "$SSH_AUTH_SOCK"   # e.g. /run/user/1000/gcr/ssh → GNOME's agent is in use
ssh-add -l              # lists keys, or "The agent has no identities."
```

If `SSH_AUTH_SOCK` is already set, just run `ssh-add`; don't start a second agent with `eval "$(ssh-agent -s)"`, or you'll end up with two agents and keys loaded in the "wrong" one. On macOS, the system agent is started by launchd, and `ssh-add --apple-use-keychain` stores the passphrase in Keychain.

---


[↑ Back to TOC](#table-of-contents)

## 10. Key Security Best Practices


[↑ Back to TOC](#table-of-contents)

### ✔ Always Use Passphrases

Unprotected keys = high risk.

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

### ✔ Protect Permissions

SSH refuses insecure keys.

Correct:

```bash
chmod 600 ~/.ssh/id_ed25519        # private key
chmod 644 ~/.ssh/id_ed25519.pub    # public key
```

See the permissions table in [section 2](#2-default-ssh-key-locations-linux) for the other files.

---


[↑ Back to TOC](#table-of-contents)

### ✔ Disable Password Authentication (Servers)

Use a drop-in file instead of editing `/etc/ssh/sshd_config` directly. On RHEL/Fedora the main file includes `/etc/ssh/sshd_config.d/*.conf` first, in alphabetical order, and **the first value sshd finds for an option wins**. The distro ships `50-redhat.conf` there, so name your file with a lower number:

```bash
sudo vi /etc/ssh/sshd_config.d/10-custom.conf
```

```ini
PasswordAuthentication no
PubkeyAuthentication yes
```

Check the syntax, confirm the effective values, then reload:

```bash
sudo sshd -t     # prints nothing if the config is valid
sudo sshd -T | grep -iE '^(passwordauthentication|pubkeyauthentication) '
sudo systemctl reload sshd
```

**Warning:**
Ensure keys work before disabling passwords. Keep your current session open and test the login from a second terminal before you log out.

---


[↑ Back to TOC](#table-of-contents)

## 11. Revoking / Removing Access

Simply delete the key line from:

```
~/.ssh/authorized_keys
```

No restart required.

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

## 13. Common Problems & Fixes

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

### ❌ Wrong Key Used

Force key:

```bash
ssh -i ~/.ssh/id_ed25519_work user@server
```

Or fix config.

---


[↑ Back to TOC](#table-of-contents)

### ❌ Server Ignores Key

On server:

```bash
sudo sshd -T | grep -i '^pubkeyauthentication'
```

Must be:

```
pubkeyauthentication yes
```

`sshd -T` prints the **effective** settings after the main file and every drop-in in `/etc/ssh/sshd_config.d/` are combined. Grepping `/etc/ssh/sshd_config` alone can mislead you: the line there may be commented out or overridden by a drop-in.

---


[↑ Back to TOC](#table-of-contents)

### ❌ SELinux Issues (RHEL Specific)

Restore contexts:

```bash
restorecon -Rv ~/.ssh
```

Very common on RHEL systems.

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

## 15. When To Generate New Keys

Generate new keys if:

* Device lost/stolen
* Private key exposed
* Role separation needed
* Periodic rotation policy

---


[↑ Back to TOC](#table-of-contents)

## 16. Good Operational Habits

- ✔ Use descriptive filenames
- ✔ Separate keys by trust boundary
- ✔ Keep config clean
- ✔ Remove unused keys
- ✔ Use passphrases always



[↑ Back to TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
