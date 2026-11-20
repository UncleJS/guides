# A Beginner's Guide to SSH

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)



## Table of Contents

- [What is SSH?](#what-is-ssh)
- [How SSH Works (The Short Version)](#how-ssh-works-the-short-version)
- [Installing SSH](#installing-ssh)
  - [Linux](#linux)
  - [macOS](#macos)
  - [Windows](#windows)
- [Connecting to a Remote Server](#connecting-to-a-remote-server)
- [Generating SSH Keys](#generating-ssh-keys)
  - [What happens next](#what-happens-next)
- [Copying Your Public Key to a Server](#copying-your-public-key-to-a-server)
  - [The easy way](#the-easy-way)
  - [The manual way](#the-manual-way)
- [The SSH Config File](#the-ssh-config-file)
  - [Common config options](#common-config-options)
- [Common SSH Commands](#common-ssh-commands)
  - [Run a single command remotely](#run-a-single-command-remotely)
  - [Copy files with SCP (Secure Copy)](#copy-files-with-scp-secure-copy)
  - [Transfer files with SFTP](#transfer-files-with-sftp)
  - [Rsync over SSH (for syncing directories)](#rsync-over-ssh-for-syncing-directories)
- [SSH Tunneling](#ssh-tunneling)
  - [Local Port Forwarding](#local-port-forwarding)
  - [Remote Port Forwarding](#remote-port-forwarding)
  - [Dynamic Port Forwarding (SOCKS Proxy)](#dynamic-port-forwarding-socks-proxy)
- [SSH Agent](#ssh-agent)
- [Securing Your SSH Server](#securing-your-ssh-server)
- [Troubleshooting](#troubleshooting)
- [Quick Reference](#quick-reference)
- [Next Steps](#next-steps)

---
## What is SSH?

SSH (Secure Shell) is a cryptographic network protocol that lets you securely connect to and control remote computers over an unsecured network. Think of it as a secure, encrypted tunnel between your computer and another machine — anything you type or receive travels through that tunnel, invisible to anyone who might be watching.

SSH replaced older, insecure protocols like Telnet and rsh, and is now the standard way to manage remote servers, transfer files, and even tunnel other traffic.

---


[↑ Back to TOC](#table-of-contents)

## How SSH Works (The Short Version)

SSH uses **public-key cryptography** to authenticate connections:

1. You have a **key pair**: a private key (secret, stays on your machine) and a public key (shared freely).
2. The remote server stores your public key.
3. When you connect, SSH proves you hold the matching private key — without ever sending it over the network.
4. Once authenticated, all traffic is encrypted.

You can also authenticate with a **password**, though key-based authentication is more secure and recommended.

---


[↑ Back to TOC](#table-of-contents)

## Installing SSH


[↑ Back to TOC](#table-of-contents)

### Linux
Most Linux distributions ship with OpenSSH pre-installed. If not:

```bash
# Debian/Ubuntu
sudo apt update && sudo apt install openssh-client

# Fedora/RHEL
sudo dnf install openssh-clients
```

To host an SSH server:

```bash
# Debian/Ubuntu
sudo apt install openssh-server

# Fedora/RHEL
sudo dnf install openssh-server
sudo systemctl enable --now sshd
```


[↑ Back to TOC](#table-of-contents)

### macOS
SSH comes pre-installed. Just open Terminal and you're ready.


[↑ Back to TOC](#table-of-contents)

### Windows
Windows 10/11 includes a built-in OpenSSH client. Enable it via:
**Settings → Apps → Optional Features → Add a feature → OpenSSH Client**

Alternatively, use [PuTTY](https://www.putty.org/) or [Windows Terminal](https://aka.ms/terminal) with WSL.

---


[↑ Back to TOC](#table-of-contents)

## Connecting to a Remote Server

The basic syntax for SSH is:

```bash
ssh username@hostname
```

**Examples:**

```bash
# Connect to a server by IP address
ssh alice@192.168.1.100

# Connect to a server by domain name
ssh alice@example.com

# Connect on a non-default port (default is 22)
ssh -p 2222 alice@example.com

# Connect with verbose output (useful for debugging)
ssh -v alice@example.com
```

On your first connection to a new server, you'll see a message like:

```
The authenticity of host 'example.com (93.184.216.34)' can't be established.
ED25519 key fingerprint is SHA256:abc123...
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Before typing `yes`, **check that the fingerprint is really the server's**. Get the real value through a trusted channel (the server's console, your admin, or your cloud provider's dashboard). On the server itself, show its host key fingerprints with:

```bash
# Run on the server (Fedora/RHEL, Debian and macOS use the same paths)
for f in /etc/ssh/ssh_host_*_key.pub; do ssh-keygen -lf "$f"; done
```

If the fingerprint matches, type `yes` to accept and save the server's key to `~/.ssh/known_hosts`. Future connections will be verified against it. If it does not match, answer `no`: someone may be intercepting the connection.

---


[↑ Back to TOC](#table-of-contents)

## Generating SSH Keys

Using SSH keys is more secure than passwords. Here's how to generate a key pair:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- `-t ed25519` specifies the key type. Ed25519 is modern and recommended.
- `-C` adds a comment (usually your email) to help identify the key.

You can also use RSA if Ed25519 isn't supported by your target system. Use at least 3072 bits (4096 is fine):

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```


[↑ Back to TOC](#table-of-contents)

### What happens next

```
Enter file in which to save the key (/home/alice/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
```

- Press **Enter** to accept the default file location.
- Enter a **passphrase** (strongly recommended — it encrypts your private key).

This creates two files:
- `~/.ssh/id_ed25519` — your **private key** (never share this)
- `~/.ssh/id_ed25519.pub` — your **public key** (safe to share)

---


[↑ Back to TOC](#table-of-contents)

## Copying Your Public Key to a Server

To enable key-based login, your public key must be added to `~/.ssh/authorized_keys` on the remote server.


[↑ Back to TOC](#table-of-contents)

### The easy way

```bash
ssh-copy-id username@hostname
```

This automatically appends your public key to the server's `authorized_keys` file.


[↑ Back to TOC](#table-of-contents)

### The manual way

```bash
# View your public key
cat ~/.ssh/id_ed25519.pub

# On the remote server, add it manually
mkdir -p ~/.ssh
echo "your-public-key-here" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys

# Fedora/RHEL (SELinux): restore the correct SELinux labels
restorecon -Rv ~/.ssh
```

> **Note:** File permissions matter. SSH will refuse to use keys if the permissions are too open.

> **SELinux note (Fedora/RHEL):** If `~/.ssh` was created by hand, copied from elsewhere, or restored from a backup, it may carry the wrong SELinux label, and `sshd` will ignore `authorized_keys` even though the permissions look right. `restorecon -Rv ~/.ssh` fixes the label. `ssh-copy-id` already does this for you. Debian/Ubuntu and macOS don't use SELinux by default, so skip this step there.

---


[↑ Back to TOC](#table-of-contents)

## The SSH Config File

Instead of typing long commands every time, you can define shortcuts in `~/.ssh/config`:

```
Host myserver
    HostName example.com
    User alice
    Port 2222
    IdentityFile ~/.ssh/id_ed25519

Host work
    HostName 10.0.1.50
    User bob
    IdentityFile ~/.ssh/work_key
```

Now you can connect with just:

```bash
ssh myserver
ssh work
```


[↑ Back to TOC](#table-of-contents)

### Common config options

| Option | Description |
|---|---|
| `HostName` | The actual address of the server |
| `User` | Username to log in as |
| `Port` | SSH port (default: 22) |
| `IdentityFile` | Path to your private key |
| `ForwardAgent` | Forward your SSH agent to the remote host (see the warning below) |
| `ProxyJump` | Connect through an intermediate (jump) host, e.g. `ProxyJump bastion` |
| `ServerAliveInterval` | Send keep-alive packets to prevent timeouts |

> **Security warning about `ForwardAgent`:** With agent forwarding on, anyone with root access on the remote host (or who has taken it over) can use your agent to log in to other servers as you while you are connected. Leave it off (the default), and never set it under `Host *`. To reach a server behind a bastion, use `ProxyJump` (or `ssh -J bastion target`) instead: your keys stay on your own machine.

---


[↑ Back to TOC](#table-of-contents)

## Common SSH Commands


[↑ Back to TOC](#table-of-contents)

### Run a single command remotely

```bash
ssh alice@example.com "ls -la /var/www"
```


[↑ Back to TOC](#table-of-contents)

### Copy files with SCP (Secure Copy)

```bash
# Copy a local file to a remote server
scp file.txt alice@example.com:/home/alice/

# Copy a remote file to your local machine
scp alice@example.com:/home/alice/file.txt ./

# Copy a directory recursively
scp -r my_folder/ alice@example.com:/home/alice/
```


[↑ Back to TOC](#table-of-contents)

### Transfer files with SFTP

```bash
sftp alice@example.com
```

Inside SFTP, you can use commands like `ls`, `cd`, `get`, `put`, `mkdir`, and `exit`.


[↑ Back to TOC](#table-of-contents)

### Rsync over SSH (for syncing directories)

```bash
rsync -avz -e ssh my_folder/ alice@example.com:/home/alice/my_folder/
```

---


[↑ Back to TOC](#table-of-contents)

## SSH Tunneling

SSH can forward ports and create encrypted tunnels — useful for accessing services that aren't publicly exposed.


[↑ Back to TOC](#table-of-contents)

### Local Port Forwarding

Forward a port on your local machine to a port on the remote server:

```bash
ssh -L 8080:localhost:80 alice@example.com
```

Now visiting `http://localhost:8080` on your machine connects to port 80 on the remote server.


[↑ Back to TOC](#table-of-contents)

### Remote Port Forwarding

Expose a local port on the remote server:

```bash
ssh -R 9090:localhost:3000 alice@example.com
```

Now port 9090 on the remote server connects back to port 3000 on your machine.


[↑ Back to TOC](#table-of-contents)

### Dynamic Port Forwarding (SOCKS Proxy)

Use SSH as a SOCKS proxy to route browser traffic:

```bash
ssh -D 1080 alice@example.com
```

Then configure your browser to use `SOCKS5 localhost:1080` as a proxy.

---


[↑ Back to TOC](#table-of-contents)

## SSH Agent

Typing your passphrase every time is inconvenient. The **SSH agent** remembers your decrypted key for the session.

```bash
# Start the agent
eval "$(ssh-agent -s)"

# Add your key (you'll enter your passphrase once)
ssh-add ~/.ssh/id_ed25519

# List loaded keys
ssh-add -l
```

On macOS, keys added with `ssh-add --apple-use-keychain` persist across reboots via Keychain.

---


[↑ Back to TOC](#table-of-contents)

## Securing Your SSH Server

If you're running an SSH server, these configuration changes significantly improve security.

**Put your changes in a drop-in file, not in `/etc/ssh/sshd_config` itself.** On Fedora/RHEL (and recent Debian/Ubuntu), the main file starts with `Include /etc/ssh/sshd_config.d/*.conf`. The files are read in alphabetical order and, for most options, **the first value sshd finds wins**. Fedora/RHEL ship `40-redhat-crypto-policies.conf` and `50-redhat.conf` there, so give your file a lower number, such as `10-custom.conf`, or its settings may be ignored. Package updates also leave your own drop-in alone.

```
# /etc/ssh/sshd_config.d/10-custom.conf

# Disable root login
PermitRootLogin no

# Disable password authentication (use keys only)
PasswordAuthentication no

# Limit login attempts
MaxAuthTries 3

# Only allow specific users
AllowUsers alice bob

# Change the default port (obscures from casual scans; see the steps below)
Port 2222
```

**Changing the port on Fedora/RHEL** needs two more steps, or sshd will fail to start (SELinux) or be unreachable (firewall):

```bash
# Allow sshd to bind the new port under SELinux
sudo semanage port -a -t ssh_port_t -p tcp 2222

# Open the new port in firewalld
sudo firewall-cmd --permanent --add-port=2222/tcp
sudo firewall-cmd --reload
```

Once you can log in on the new port, you can close the old one with `sudo firewall-cmd --permanent --remove-service=ssh && sudo firewall-cmd --reload`. On Debian/Ubuntu with `ufw`, use `sudo ufw allow 2222/tcp` instead (no SELinux step).

**Ciphers and key exchange (`KexAlgorithms`, `Ciphers`, `MACs`):** On Fedora/RHEL these are set system-wide by **crypto-policies** (loaded through `40-redhat-crypto-policies.conf`). Rather than listing algorithms in sshd's config, change the policy, which also covers other programs:

```bash
update-crypto-policies --show          # current policy, e.g. DEFAULT
sudo update-crypto-policies --set FUTURE   # stricter policy (then restart sshd)
```

If you do set `KexAlgorithms` or `Ciphers` yourself, the line only takes effect from a drop-in that sorts before `40-redhat-crypto-policies.conf`, and it overrides the system policy for sshd. On Debian/Ubuntu and macOS there are no crypto-policies; the OpenSSH defaults are already modern.

**Check and apply the configuration:**

```bash
# Check the syntax; prints nothing if the config is valid
sudo sshd -t

# Show the effective settings after all files are combined
sudo sshd -T | grep -iE '^(permitrootlogin|passwordauthentication|port|kexalgorithms) '

# Restart the SSH service (Debian/Ubuntu: the unit is called "ssh")
sudo systemctl restart sshd
```

> **Warning:** Keep your current SSH session open while you make and test these changes. Open a **second** terminal and log in again (on the new port, with your key) before you close the first one. If you've made a mistake, you can still fix it from the session that is still open. Before disabling password authentication, make sure your key-based login works, or you may lock yourself out.

---


[↑ Back to TOC](#table-of-contents)

## Troubleshooting

**"Permission denied (publickey)"**
- Ensure your public key is in `~/.ssh/authorized_keys` on the server.
- Check file permissions: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`.
- On Fedora/RHEL, fix SELinux labels: `restorecon -Rv ~/.ssh`.
- Verify you're using the right key with `ssh -i ~/.ssh/your_key user@host`.

**"Connection refused"**
- The SSH service may not be running: `sudo systemctl start sshd`.
- A firewall may be blocking port 22. On Fedora/RHEL (firewalld): `sudo firewall-cmd --permanent --add-service=ssh && sudo firewall-cmd --reload`. On Debian/Ubuntu (ufw): `sudo ufw allow ssh`.

**"Host key verification failed"**
- The server's fingerprint changed (could indicate a security issue or the server was reinstalled).
- Remove the old entry: `ssh-keygen -R hostname` and reconnect.

**Slow or dropping connections**
Add to your `~/.ssh/config`:
```
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

**Debug with verbose mode**
```bash
ssh -vvv alice@example.com
```

---


[↑ Back to TOC](#table-of-contents)

## Quick Reference

| Command | Description |
|---|---|
| `ssh user@host` | Connect to a remote host |
| `ssh -p 2222 user@host` | Connect on a specific port |
| `ssh-keygen -t ed25519` | Generate an Ed25519 key pair |
| `ssh-copy-id user@host` | Copy your public key to a server |
| `scp file.txt user@host:~/` | Copy a file to a remote server |
| `scp user@host:~/file.txt ./` | Download a file from a server |
| `sftp user@host` | Start an interactive SFTP session |
| `ssh-add ~/.ssh/id_ed25519` | Add a key to the SSH agent |
| `ssh -L 8080:localhost:80 user@host` | Local port forwarding |
| `ssh-keygen -R hostname` | Remove a host from known_hosts |

---


[↑ Back to TOC](#table-of-contents)

## Next Steps

Now that you have the basics down, here are some things to explore:

- **Multiplexing** — reuse a single connection for multiple sessions (`ControlMaster auto`)
- **Jump hosts / Bastion servers** — hop through an intermediate server with `ProxyJump`
- **Certificate-based authentication** — more scalable than managing individual public keys
- **Ansible and other automation tools** — built on top of SSH for server management at scale
- **Mosh** — a more resilient alternative to SSH for unstable connections

SSH is one of the most powerful tools in a developer's or sysadmin's toolkit. With these fundamentals, you're well on your way to managing remote systems confidently and securely.

[↑ Back to TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
