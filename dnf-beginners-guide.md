# DNF Package Manager: A Beginner's Guide

DNF (Dandified YUM) is the default package manager for RPM-based Linux distributions such as Fedora, RHEL (Red Hat Enterprise Linux), CentOS Stream, and AlmaLinux. It handles installing, updating, removing, and querying software packages, along with automatically resolving dependencies.

---

## Table of Contents

1. [What is DNF?](#what-is-dnf)
2. [Basic Concepts](#basic-concepts)
3. [Getting Started](#getting-started)
4. [Installing Packages](#installing-packages)
5. [Removing Packages](#removing-packages)
6. [Updating Packages](#updating-packages)
7. [Searching for Packages](#searching-for-packages)
8. [Getting Package Information](#getting-package-information)
9. [Managing Repositories](#managing-repositories)
10. [Working with Package Groups](#working-with-package-groups)
11. [History and Rollback](#history-and-rollback)
12. [Cleaning Up](#cleaning-up)
13. [Useful Flags and Options](#useful-flags-and-options)
14. [Configuration](#configuration)
15. [Common Recipes](#common-recipes)
16. [Tips and Best Practices](#tips-and-best-practices)

---

## What is DNF?

DNF replaced the older YUM (Yellowdog Updater Modified) package manager starting with Fedora 22. It is faster, uses less memory, and has a cleaner API. Under the hood, DNF:

- Resolves package **dependencies** automatically
- Downloads packages from **repositories** (remote or local collections of software)
- Verifies **GPG signatures** to ensure package integrity
- Maintains a **transaction history** so you can undo changes

---

## Basic Concepts

**Package** — A compressed archive (`.rpm` file) containing software, metadata, and installation instructions.

**Repository (repo)** — A server or directory hosting a collection of packages with an index DNF can read. Repos are defined in `/etc/yum.repos.d/`.

**Dependency** — A package that another package requires to function. DNF resolves these for you automatically.

**Transaction** — A set of operations (installs, updates, removals) applied together. If any step fails, the whole transaction is rolled back.

**Epoch, Version, Release (EVR)** — The three-part version string used to compare packages: `epoch:version-release.arch` (e.g., `2:5.14.0-284.fc38.x86_64`).

---

## Getting Started

DNF is pre-installed on supported distributions. Most operations require root/sudo privileges.

Check that DNF is available and see its version:

```bash
dnf --version
```

Display the built-in help:

```bash
dnf help
dnf help <command>   # help for a specific command
```

---

## Installing Packages

### Install a single package

```bash
sudo dnf install <package-name>
```

Example:

```bash
sudo dnf install vim
```

DNF will show you a summary of what will be installed (including dependencies) and ask for confirmation. Type `y` and press Enter to proceed.

### Install multiple packages at once

```bash
sudo dnf install vim git curl wget
```

### Install a local `.rpm` file

```bash
sudo dnf install ./mypackage.rpm
```

Using `dnf install` instead of `rpm -i` for local files ensures dependencies are resolved from your repos.

### Install a specific version

```bash
sudo dnf install vim-9.0.1234
```

### Reinstall a package

Useful if package files are corrupted:

```bash
sudo dnf reinstall vim
```

---

## Removing Packages

### Remove a single package

```bash
sudo dnf remove <package-name>
```

Example:

```bash
sudo dnf remove vim
```

### Remove a package and its unused dependencies

DNF automatically marks dependencies as "auto-installed." When you remove a package, you can clean up orphaned dependencies with:

```bash
sudo dnf autoremove
```

Or combine both steps:

```bash
sudo dnf remove vim && sudo dnf autoremove
```

> ⚠️ **Caution:** Be careful when removing packages — DNF may also remove packages that depend on the one you're removing. Always review the transaction summary before confirming.

---

## Updating Packages

### Update all packages

```bash
sudo dnf update
```

This fetches the latest package metadata and upgrades all installed packages with available updates.

### Update a specific package

```bash
sudo dnf update vim
```

### Check for available updates without installing

```bash
dnf check-update
```

### Upgrade vs Update

`dnf upgrade` and `dnf update` are functionally identical in modern DNF. Both handle version upgrades and obsoletes.

### Upgrade your entire distribution (e.g., Fedora release upgrade)

```bash
sudo dnf system-upgrade download --releasever=<version>
sudo dnf system-upgrade reboot
```

---

## Searching for Packages

### Search by name or summary

```bash
dnf search <keyword>
```

Example:

```bash
dnf search video editor
```

### Search in package names only

```bash
dnf search --name <keyword>
```

### Find which package provides a specific file or command

```bash
dnf provides <file-or-command>
```

Example — find what provides the `curl` command:

```bash
dnf provides curl
```

Find what package owns a specific file path:

```bash
dnf provides /usr/bin/python3
```

---

## Getting Package Information

### Show detailed package info

```bash
dnf info <package-name>
```

Displays version, description, size, repo, license, and more.

### List all installed packages

```bash
dnf list installed
```

### List all available packages

```bash
dnf list available
```

### List packages with available updates

```bash
dnf list updates
```

### Check if a specific package is installed

```bash
dnf list installed vim
```

### List files installed by a package

```bash
dnf repoquery -l <package-name>
```

### Show dependencies of a package

```bash
dnf repoquery --requires <package-name>
```

### Show what packages depend on a given package

```bash
dnf repoquery --whatrequires <package-name>
```

---

## Managing Repositories

### List all enabled repositories

```bash
dnf repolist
```

### List all repositories (enabled and disabled)

```bash
dnf repolist all
```

### Enable a repository temporarily (for one command)

```bash
sudo dnf install <package> --enablerepo=<repo-id>
```

### Enable a repository permanently

```bash
sudo dnf config-manager --enable <repo-id>
```

### Disable a repository permanently

```bash
sudo dnf config-manager --disable <repo-id>
```

### Add a new repository

```bash
sudo dnf config-manager --add-repo <repo-url>
```

Or place a `.repo` file manually in `/etc/yum.repos.d/`:

```ini
# /etc/yum.repos.d/myrepo.repo
[myrepo]
name=My Custom Repo
baseurl=https://example.com/repo/
enabled=1
gpgcheck=1
gpgkey=https://example.com/repo/RPM-GPG-KEY
```

### Enable Third-Party Repos (Fedora)

Popular third-party repos include **RPM Fusion**, which provides software not included in Fedora by default (e.g., media codecs, proprietary drivers):

```bash
sudo dnf install \
  https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

---

## Working with Package Groups

Groups are collections of related packages bundled together for convenience.

### List all available groups

```bash
dnf group list
```

### Get info about a group

```bash
dnf group info "Development Tools"
```

### Install a group

```bash
sudo dnf group install "Development Tools"
```

### Remove a group

```bash
sudo dnf group remove "Development Tools"
```

### Update all packages in a group

```bash
sudo dnf group update "Development Tools"
```

---

## History and Rollback

DNF keeps a log of all transactions, allowing you to view and undo changes.

### View transaction history

```bash
dnf history
```

### View details of a specific transaction

```bash
dnf history info <transaction-id>
```

### Undo a specific transaction

```bash
sudo dnf history undo <transaction-id>
```

### Redo a specific transaction

```bash
sudo dnf history redo <transaction-id>
```

### Roll back to a specific transaction

```bash
sudo dnf history rollback <transaction-id>
```

This reverts your system to the state it was in after the given transaction ID.

---

## Cleaning Up

Over time, DNF caches metadata and downloaded packages. Cleaning up can free disk space.

### Remove cached package data

```bash
sudo dnf clean packages
```

### Remove cached metadata

```bash
sudo dnf clean metadata
```

### Remove all cached data

```bash
sudo dnf clean all
```

### Remove orphaned packages (auto-installed, no longer needed)

```bash
sudo dnf autoremove
```

### Rebuild the metadata cache

```bash
sudo dnf makecache
```

---

## Useful Flags and Options

| Flag | Description |
|------|-------------|
| `-y` | Automatically answer "yes" to all prompts |
| `-q` | Quiet mode — suppress most output |
| `--nogpgcheck` | Skip GPG signature verification (not recommended) |
| `--enablerepo=<id>` | Enable a repo for this command only |
| `--disablerepo=<id>` | Disable a repo for this command only |
| `--exclude=<pkg>` | Exclude a package from the operation |
| `--best` | Always try to install the best available version |
| `--allowerasing` | Allow removing packages to resolve conflicts |
| `--downloadonly` | Download packages without installing |
| `--downloaddir=<path>` | Specify directory for `--downloadonly` |
| `--setopt=<key=value>` | Override a config option temporarily |

### Example: Non-interactive update

```bash
sudo dnf update -y
```

### Example: Download a package without installing it

```bash
sudo dnf install --downloadonly --downloaddir=/tmp vim
```

---

## Configuration

DNF's main configuration file is `/etc/dnf/dnf.conf`. Here are some useful settings:

```ini
[main]
gpgcheck=1              # Always verify GPG signatures
installonly_limit=3     # Keep only the 3 most recent kernel versions
clean_requirements_on_remove=True  # Auto-remove unused deps on package removal
best=True               # Always try to install the latest version
skip_if_unavailable=False  # Fail if a repo is unavailable
fastestmirror=True      # Choose the fastest mirror automatically
max_parallel_downloads=5   # Download up to 5 packages simultaneously
```

You can also override any setting temporarily on the command line:

```bash
sudo dnf update --setopt=max_parallel_downloads=10
```

---

## Common Recipes

### Install development tools

```bash
sudo dnf group install "Development Tools"
sudo dnf install gcc make cmake python3-devel
```

### Find and install a missing library

```bash
dnf provides libssl.so.3
sudo dnf install openssl-libs
```

### Keep your system up to date automatically (Fedora)

```bash
sudo dnf install dnf-automatic
sudo systemctl enable --now dnf-automatic.timer
```

Edit `/etc/dnf/automatic.conf` to configure behavior (e.g., download only vs. auto-apply updates).

### Downgrade a package to a previous version

```bash
sudo dnf downgrade vim
```

### Install a package from a different release (Fedora Rawhide, etc.)

```bash
sudo dnf install --releasever=rawhide <package>
```

### Lock a package to its current version

```bash
sudo dnf install python3-dnf-plugin-versionlock
sudo dnf versionlock add vim
```

---

## Tips and Best Practices

**Always review the transaction summary.** Before typing `y`, read what DNF plans to install, remove, or update. Surprises are rare but possible.

**Use `dnf check-update` before updating.** It lets you see what's pending without committing to anything.

**Keep your cache fresh.** Run `sudo dnf makecache` periodically so package searches and installs are faster.

**Prefer `dnf install` over `rpm -i`.** Even for local `.rpm` files, DNF resolves dependencies from your repos automatically.

**Audit history after problems.** If something broke after an update, `dnf history` and `dnf history undo` are your friends.

**Use `--best --allowerasing` to resolve stubborn conflicts.** This tells DNF it can replace conflicting packages to find a solution:

```bash
sudo dnf update --best --allowerasing
```

**Check enabled repos before troubleshooting.** A missing package is often in a disabled or un-added repo. Run `dnf repolist` to check.

**Be cautious with `--nogpgcheck`.** It disables a critical security check. Only use it for trusted local packages during testing.

---

## Quick Reference Cheat Sheet

```
dnf install <pkg>          Install a package
dnf remove <pkg>           Remove a package
dnf update                 Update all packages
dnf update <pkg>           Update a specific package
dnf search <keyword>       Search for packages
dnf info <pkg>             Show package details
dnf provides <file>        Find which package owns a file
dnf list installed         List installed packages
dnf list updates           List available updates
dnf repolist               Show enabled repos
dnf history                Show transaction history
dnf history undo <id>      Undo a transaction
dnf autoremove             Remove orphaned packages
dnf clean all              Clear all cached data
dnf group list             List package groups
dnf group install <group>  Install a package group
```

---

*This guide covers DNF as used on Fedora, RHEL 8+, CentOS Stream, AlmaLinux, and Rocky Linux. Some behavior may vary slightly between distributions.*
