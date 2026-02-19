# A Beginner's Guide to SSH

## What is SSH?

SSH (Secure Shell) is a cryptographic network protocol that lets you securely connect to and control remote computers over an unsecured network. Think of it as a secure, encrypted tunnel between your computer and another machine — anything you type or receive travels through that tunnel, invisible to anyone who might be watching.

SSH replaced older, insecure protocols like Telnet and rsh, and is now the standard way to manage remote servers, transfer files, and even tunnel other traffic.

---

## How SSH Works (The Short Version)

SSH uses **public-key cryptography** to authenticate connections:

1. You have a **key pair**: a private key (secret, stays on your machine) and a public key (shared freely).
2. The remote server stores your public key.
3. When you connect, SSH proves you hold the matching private key — without ever sending it over the network.
4. Once authenticated, all traffic is encrypted.

You can also authenticate with a **password**, though key-based authentication is more secure and recommended.

---

## Installing SSH

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

### macOS
SSH comes pre-installed. Just open Terminal and you're ready.

### Windows
Windows 10/11 includes a built-in OpenSSH client. Enable it via:
**Settings → Apps → Optional Features → Add a feature → OpenSSH Client**

Alternatively, use [PuTTY](https://www.putty.org/) or [Windows Terminal](https://aka.ms/terminal) with WSL.

---

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

Type `yes` to accept and save the server's fingerprint to `~/.ssh/known_hosts`. Future connections will be verified against this fingerprint.

---

## Generating SSH Keys

Using SSH keys is more secure than passwords. Here's how to generate a key pair:

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

- `-t ed25519` specifies the key type. Ed25519 is modern and recommended.
- `-C` adds a comment (usually your email) to help identify the key.

You can also use RSA if Ed25519 isn't supported by your target system:

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

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

## Copying Your Public Key to a Server

To enable key-based login, your public key must be added to `~/.ssh/authorized_keys` on the remote server.

### The easy way

```bash
ssh-copy-id username@hostname
```

This automatically appends your public key to the server's `authorized_keys` file.

### The manual way

```bash
# View your public key
cat ~/.ssh/id_ed25519.pub

# On the remote server, add it manually
mkdir -p ~/.ssh
echo "your-public-key-here" >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

> **Note:** File permissions matter. SSH will refuse to use keys if the permissions are too open.

---

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

### Common config options

| Option | Description |
|---|---|
| `HostName` | The actual address of the server |
| `User` | Username to log in as |
| `Port` | SSH port (default: 22) |
| `IdentityFile` | Path to your private key |
| `ForwardAgent` | Forward your SSH agent to the remote host |
| `ServerAliveInterval` | Send keep-alive packets to prevent timeouts |

---

## Common SSH Commands

### Run a single command remotely

```bash
ssh alice@example.com "ls -la /var/www"
```

### Copy files with SCP (Secure Copy)

```bash
# Copy a local file to a remote server
scp file.txt alice@example.com:/home/alice/

# Copy a remote file to your local machine
scp alice@example.com:/home/alice/file.txt ./

# Copy a directory recursively
scp -r my_folder/ alice@example.com:/home/alice/
```

### Transfer files with SFTP

```bash
sftp alice@example.com
```

Inside SFTP, you can use commands like `ls`, `cd`, `get`, `put`, `mkdir`, and `exit`.

### Rsync over SSH (for syncing directories)

```bash
rsync -avz -e ssh my_folder/ alice@example.com:/home/alice/my_folder/
```

---

## SSH Tunneling

SSH can forward ports and create encrypted tunnels — useful for accessing services that aren't publicly exposed.

### Local Port Forwarding

Forward a port on your local machine to a port on the remote server:

```bash
ssh -L 8080:localhost:80 alice@example.com
```

Now visiting `http://localhost:8080` on your machine connects to port 80 on the remote server.

### Remote Port Forwarding

Expose a local port on the remote server:

```bash
ssh -R 9090:localhost:3000 alice@example.com
```

Now port 9090 on the remote server connects back to port 3000 on your machine.

### Dynamic Port Forwarding (SOCKS Proxy)

Use SSH as a SOCKS proxy to route browser traffic:

```bash
ssh -D 1080 alice@example.com
```

Then configure your browser to use `SOCKS5 localhost:1080` as a proxy.

---

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

## Securing Your SSH Server

If you're running an SSH server, these configuration changes (in `/etc/ssh/sshd_config`) significantly improve security:

```
# Disable root login
PermitRootLogin no

# Disable password authentication (use keys only)
PasswordAuthentication no

# Limit login attempts
MaxAuthTries 3

# Only allow specific users
AllowUsers alice bob

# Change the default port (obscures from casual scans)
Port 2222

# Use modern key exchange algorithms only
KexAlgorithms curve25519-sha256,diffie-hellman-group14-sha256
```

After making changes, restart the SSH service:

```bash
sudo systemctl restart sshd
```

> **Warning:** Before disabling password authentication, make sure your key-based login works. Otherwise you may lock yourself out.

---

## Troubleshooting

**"Permission denied (publickey)"**
- Ensure your public key is in `~/.ssh/authorized_keys` on the server.
- Check file permissions: `chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`.
- Verify you're using the right key with `ssh -i ~/.ssh/your_key user@host`.

**"Connection refused"**
- The SSH service may not be running: `sudo systemctl start sshd`.
- A firewall may be blocking port 22: `sudo ufw allow ssh`.

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

## Next Steps

Now that you have the basics down, here are some things to explore:

- **Multiplexing** — reuse a single connection for multiple sessions (`ControlMaster auto`)
- **Jump hosts / Bastion servers** — hop through an intermediate server with `ProxyJump`
- **Certificate-based authentication** — more scalable than managing individual public keys
- **Ansible and other automation tools** — built on top of SSH for server management at scale
- **Mosh** — a more resilient alternative to SSH for unstable connections

SSH is one of the most powerful tools in a developer's or sysadmin's toolkit. With these fundamentals, you're well on your way to managing remote systems confidently and securely.
