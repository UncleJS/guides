# Netstat: A Beginner's Guide

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)



## Table of Contents

- [What is Netstat?](#what-is-netstat)
- [Installation & Availability](#installation-availability)
- [Basic Syntax](#basic-syntax)
- [Understanding the Output](#understanding-the-output)
- [Connection States Explained](#connection-states-explained)
- [Essential Flags & Options](#essential-flags-options)
  - [Linux / macOS](#linux-macos)
  - [Windows](#windows)
- [Common Commands with Examples](#common-commands-with-examples)
  - [1. Show all active connections and listening ports](#1-show-all-active-connections-and-listening-ports)
  - [2. Show listening ports only (what's waiting for connections)](#2-show-listening-ports-only-whats-waiting-for-connections)
  - [3. Show numeric addresses (no DNS lookup)](#3-show-numeric-addresses-no-dns-lookup)
  - [4. Show which program is using which port](#4-show-which-program-is-using-which-port)
  - [5. Show the routing table](#5-show-the-routing-table)
  - [6. Show network interface statistics](#6-show-network-interface-statistics)
  - [7. Show protocol statistics](#7-show-protocol-statistics)
  - [8. Watch connections in real time](#8-watch-connections-in-real-time)
  - [9. Filter output with grep (Linux/macOS)](#9-filter-output-with-grep-linuxmacos)
  - [10. Windows — Find which app owns a port](#10-windows-find-which-app-owns-a-port)
- [Practical Use Cases](#practical-use-cases)
  - [Is my web server running?](#is-my-web-server-running)
  - [Why is my connection slow?](#why-is-my-connection-slow)
  - [Is something connecting to an unexpected address?](#is-something-connecting-to-an-unexpected-address)
  - [Check for a port conflict before starting a service](#check-for-a-port-conflict-before-starting-a-service)
  - [How many connections does my server have?](#how-many-connections-does-my-server-have)
- [Netstat vs. Modern Alternatives](#netstat-vs-modern-alternatives)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
- [Tips for Beginners](#tips-for-beginners)

---
## What is Netstat?

`netstat` (network statistics) is a command-line tool that displays information about your computer's network connections, routing tables, interface statistics, and more. It's one of the most useful tools for diagnosing network issues, identifying open ports, and monitoring active connections — and it's available on Linux, macOS, and Windows.

---


[↑ Goto TOC](#table-of-contents)

## Installation & Availability

| OS | Status |
|---|---|
| Linux | Usually pre-installed. If not: `sudo apt install net-tools` (Debian/Ubuntu) or `sudo yum install net-tools` (RHEL/CentOS) |
| macOS | Pre-installed |
| Windows | Pre-installed (run via Command Prompt or PowerShell) |

> **Note:** On modern Linux systems, `netstat` is considered legacy and is part of the `net-tools` package. The recommended modern replacement is `ss` (socket statistics), but `netstat` remains widely used and understood.

---


[↑ Goto TOC](#table-of-contents)

## Basic Syntax

```
netstat [options]
```

Running `netstat` with no options displays a list of active connections.

---


[↑ Goto TOC](#table-of-contents)

## Understanding the Output

Before diving into flags, it helps to understand what `netstat` shows you by default. Here's a sample output:

```
Proto  Recv-Q  Send-Q  Local Address          Foreign Address        State
tcp       0       0    192.168.1.5:54321      142.250.80.46:443      ESTABLISHED
tcp       0       0    0.0.0.0:22             0.0.0.0:*              LISTEN
udp       0       0    0.0.0.0:68             0.0.0.0:*
```

**Column breakdown:**

- **Proto** — The protocol in use (`tcp`, `udp`, `unix` for Unix sockets)
- **Recv-Q** — Data received but not yet read by the application
- **Send-Q** — Data sent but not yet acknowledged by the remote host
- **Local Address** — Your machine's IP and port (the left side of the connection)
- **Foreign Address** — The remote machine's IP and port
- **State** — The current state of the connection (TCP only)

---


[↑ Goto TOC](#table-of-contents)

## Connection States Explained

Understanding TCP states is crucial for reading netstat output:

| State | Meaning |
|---|---|
| `LISTEN` | The port is open and waiting for incoming connections |
| `ESTABLISHED` | An active, live connection |
| `TIME_WAIT` | Connection is closed but waiting to ensure all packets are received |
| `CLOSE_WAIT` | Remote end closed the connection; waiting for local app to close |
| `SYN_SENT` | Your machine is attempting to establish a connection |
| `SYN_RECV` | A connection request has been received |
| `FIN_WAIT1` / `FIN_WAIT2` | Connection is in the process of closing |
| `CLOSED` | Connection is fully closed |

---


[↑ Goto TOC](#table-of-contents)

## Essential Flags & Options


[↑ Goto TOC](#table-of-contents)

### Linux / macOS

| Flag | Description |
|---|---|
| `-a` | Show **all** connections and listening ports |
| `-t` | Show only **TCP** connections |
| `-u` | Show only **UDP** connections |
| `-l` | Show only **listening** ports |
| `-n` | Show **numeric** addresses (skip DNS resolution — faster) |
| `-p` | Show the **PID and program name** using each socket |
| `-r` | Show the **routing table** |
| `-i` | Show **network interface** statistics |
| `-s` | Show **summary statistics** per protocol |
| `-c` | **Continuously** refresh the output |
| `-e` | Show **extended** information |


[↑ Goto TOC](#table-of-contents)

### Windows

| Flag | Description |
|---|---|
| `-a` | Show all connections and listening ports |
| `-n` | Show numeric addresses |
| `-o` | Show the **owning process ID** (PID) |
| `-b` | Show the **executable** involved in each connection (requires admin) |
| `-p proto` | Filter by protocol (e.g., `-p tcp`) |
| `-r` | Show the routing table |
| `-s` | Show per-protocol statistics |
| `5` | Refresh every 5 seconds (append a number for interval) |

---


[↑ Goto TOC](#table-of-contents)

## Common Commands with Examples


[↑ Goto TOC](#table-of-contents)

### 1. Show all active connections and listening ports

```bash
netstat -a
```

This is the most common starting point. It shows every open socket on your machine.

---


[↑ Goto TOC](#table-of-contents)

### 2. Show listening ports only (what's waiting for connections)

```bash
# Linux/macOS
netstat -l

# More specific — only listening TCP ports
netstat -lt
```

This is useful for seeing which services are running and exposed on your machine.

---


[↑ Goto TOC](#table-of-contents)

### 3. Show numeric addresses (no DNS lookup)

```bash
netstat -an
```

By default, netstat tries to resolve IP addresses to hostnames, which is slow. Adding `-n` skips this and gives you raw IPs and port numbers instantly.

---


[↑ Goto TOC](#table-of-contents)

### 4. Show which program is using which port

```bash
# Linux (requires sudo for all processes)
sudo netstat -tulnp

# Windows (requires admin)
netstat -ano
```

The output will include a PID column. On Linux, you'll also see the program name directly. On Windows, use Task Manager or `tasklist /fi "PID eq <PID>"` to look up the process.

**Breaking down `tulnp`:**
- `t` = TCP
- `u` = UDP
- `l` = Listening
- `n` = Numeric
- `p` = Show PID/program

This is the single most useful `netstat` command for most people.

---


[↑ Goto TOC](#table-of-contents)

### 5. Show the routing table

```bash
netstat -r
```

This shows how your machine decides where to send network traffic — similar to `route -n` on Linux or `route print` on Windows.

---


[↑ Goto TOC](#table-of-contents)

### 6. Show network interface statistics

```bash
netstat -i
```

Displays statistics per interface (e.g., `eth0`, `wlan0`) including packets sent/received and errors.

---


[↑ Goto TOC](#table-of-contents)

### 7. Show protocol statistics

```bash
netstat -s
```

Gives a breakdown of statistics for TCP, UDP, ICMP, and IP — useful for spotting errors or dropped packets at a protocol level.

---


[↑ Goto TOC](#table-of-contents)

### 8. Watch connections in real time

```bash
# Linux/macOS — refresh every second
netstat -an -c

# Windows — refresh every 2 seconds
netstat -an 2
```

---


[↑ Goto TOC](#table-of-contents)

### 9. Filter output with grep (Linux/macOS)

```bash
# Show only ESTABLISHED connections
netstat -an | grep ESTABLISHED

# Check if port 80 is in use
netstat -an | grep ':80'

# Find what's listening on port 443
netstat -tlnp | grep ':443'
```

---


[↑ Goto TOC](#table-of-contents)

### 10. Windows — Find which app owns a port

```bash
# Step 1: Find the PID using the port
netstat -ano | findstr :8080

# Step 2: Look up the PID (e.g., PID 1234)
tasklist /fi "PID eq 1234"
```

---


[↑ Goto TOC](#table-of-contents)

## Practical Use Cases


[↑ Goto TOC](#table-of-contents)

### Is my web server running?

```bash
netstat -tlnp | grep ':80\|:443'
```

If you see a process `LISTEN`ing on port 80 or 443, your web server is up.

---


[↑ Goto TOC](#table-of-contents)

### Why is my connection slow?

```bash
netstat -s | grep -i retransmit
```

A high number of retransmitted segments suggests packet loss or a poor connection.

---


[↑ Goto TOC](#table-of-contents)

### Is something connecting to an unexpected address?

```bash
netstat -an | grep ESTABLISHED
```

Review the foreign addresses. Unexpected IPs in the list could indicate unwanted outbound connections.

---


[↑ Goto TOC](#table-of-contents)

### Check for a port conflict before starting a service

```bash
netstat -tlnp | grep ':3000'
```

If something is already listening on port 3000, your new app will fail to bind to it.

---


[↑ Goto TOC](#table-of-contents)

### How many connections does my server have?

```bash
netstat -an | grep ESTABLISHED | wc -l
```

---


[↑ Goto TOC](#table-of-contents)

## Netstat vs. Modern Alternatives

On Linux, `netstat` is deprecated in favor of `ss`, which is faster and more feature-rich.

| Task | netstat | ss equivalent |
|---|---|---|
| All sockets | `netstat -a` | `ss -a` |
| TCP listening | `netstat -tln` | `ss -tln` |
| With PIDs | `netstat -tlnp` | `ss -tlnp` |
| UDP | `netstat -uln` | `ss -uln` |

The flags are largely compatible, so learning `netstat` translates almost directly to using `ss`.

---


[↑ Goto TOC](#table-of-contents)

## Quick Reference Cheat Sheet

```bash
# The all-in-one command — show TCP/UDP, listening, numeric, with PIDs
sudo netstat -tulnp

# All active connections, numeric
netstat -an

# Routing table
netstat -r

# Interface stats
netstat -i

# Protocol stats
netstat -s

# Real-time monitoring
netstat -c

# Find who owns port 8080
sudo netstat -tlnp | grep ':8080'

# Count established connections
netstat -an | grep ESTABLISHED | wc -l
```

---


[↑ Goto TOC](#table-of-contents)

## Tips for Beginners

**Start with `sudo netstat -tulnp`.** This single command tells you almost everything you need: what protocols, what ports, what IPs, and what programs are involved. It's the Swiss Army knife of netstat commands.

**Use `-n` almost always.** DNS resolution slows netstat down considerably. Get used to looking up IPs manually if needed.

**Don't panic about `TIME_WAIT`.** Seeing many `TIME_WAIT` connections is normal on busy servers. It means connections are closing gracefully.

**On Windows, combine with `tasklist`.** Since Windows doesn't show program names in netstat by default, the `netstat -ano` + `tasklist` combo is your best friend.

**Graduate to `ss` on Linux.** Once you're comfortable with `netstat`, try `ss` — the syntax is nearly identical but it's faster and more powerful.

---

*That's everything you need to get started with `netstat`. It's a simple tool with a lot of depth — the more you use it, the more natural reading network output becomes.*

[↑ Goto TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
