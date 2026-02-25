# DNF Package Manager: A Beginner's Guide

DNF (Dandified YUM) is the default package manager for RPM-based Linux distributions such as Fedora, RHEL (Red Hat Enterprise Linux), CentOS Stream, and AlmaLinux. It handles installing, updating, removing, and querying software packages, along with automatically resolving dependencies.


## Table of Contents

- [What is DNF?](#what-is-dnf)
- [Basic Concepts](#basic-concepts)
- [Getting Started](#getting-started)
- [Installing Packages](#installing-packages)
  - [Install a single package](#install-a-single-package)
  - [Install multiple packages at once](#install-multiple-packages-at-once)
  - [Install a local `.rpm` file](#install-a-local-rpm-file)
  - [Install a specific version](#install-a-specific-version)
  - [Reinstall a package](#reinstall-a-package)
- [Removing Packages](#removing-packages)
  - [Remove a single package](#remove-a-single-package)
  - [Remove a package and its unused dependencies](#remove-a-package-and-its-unused-dependencies)
- [Updating Packages](#updating-packages)
  - [Update all packages](#update-all-packages)
  - [Update a specific package](#update-a-specific-package)
  - [Check for available updates without installing](#check-for-available-updates-without-installing)
  - [Upgrade vs Update](#upgrade-vs-update)
  - [Upgrade your entire distribution (e.g., Fedora release upgrade)](#upgrade-your-entire-distribution-eg-fedora-release-upgrade)
- [Searching for Packages](#searching-for-packages)
  - [Search by name or summary](#search-by-name-or-summary)
  - [Search in package names only](#search-in-package-names-only)
  - [Find which package provides a specific file or command](#find-which-package-provides-a-specific-file-or-command)
- [Getting Package Information](#getting-package-information)
  - [Show detailed package info](#show-detailed-package-info)
  - [List all installed packages](#list-all-installed-packages)
  - [List all available packages](#list-all-available-packages)
  - [List packages with available updates](#list-packages-with-available-updates)
  - [Check if a specific package is installed](#check-if-a-specific-package-is-installed)
  - [List files installed by a package](#list-files-installed-by-a-package)
  - [Show dependencies of a package](#show-dependencies-of-a-package)
  - [Show what packages depend on a given package](#show-what-packages-depend-on-a-given-package)
- [Managing Repositories](#managing-repositories)
  - [List all enabled repositories](#list-all-enabled-repositories)
  - [List all repositories (enabled and disabled)](#list-all-repositories-enabled-and-disabled)
  - [Enable a repository temporarily (for one command)](#enable-a-repository-temporarily-for-one-command)
  - [Enable a repository permanently](#enable-a-repository-permanently)
  - [Disable a repository permanently](#disable-a-repository-permanently)
  - [Add a new repository](#add-a-new-repository)
  - [Enable Third-Party Repos (Fedora)](#enable-third-party-repos-fedora)
- [Working with Package Groups](#working-with-package-groups)
  - [List all available groups](#list-all-available-groups)
  - [Get info about a group](#get-info-about-a-group)
  - [Install a group](#install-a-group)
  - [Remove a group](#remove-a-group)
  - [Update all packages in a group](#update-all-packages-in-a-group)
- [History and Rollback](#history-and-rollback)
  - [View transaction history](#view-transaction-history)
  - [View details of a specific transaction](#view-details-of-a-specific-transaction)
  - [Undo a specific transaction](#undo-a-specific-transaction)
  - [Redo a specific transaction](#redo-a-specific-transaction)
  - [Roll back to a specific transaction](#roll-back-to-a-specific-transaction)
- [Cleaning Up](#cleaning-up)
  - [Remove cached package data](#remove-cached-package-data)
  - [Remove cached metadata](#remove-cached-metadata)
  - [Remove all cached data](#remove-all-cached-data)
  - [Remove orphaned packages (auto-installed, no longer needed)](#remove-orphaned-packages-auto-installed-no-longer-needed)
  - [Rebuild the metadata cache](#rebuild-the-metadata-cache)
- [Useful Flags and Options](#useful-flags-and-options)
  - [Example: Non-interactive update](#example-non-interactive-update)
  - [Example: Download a package without installing it](#example-download-a-package-without-installing-it)
- [Configuration](#configuration)
- [Common Recipes](#common-recipes)
  - [Install development tools](#install-development-tools)
  - [Find and install a missing library](#find-and-install-a-missing-library)
  - [Keep your system up to date automatically (Fedora)](#keep-your-system-up-to-date-automatically-fedora)
  - [Downgrade a package to a previous version](#downgrade-a-package-to-a-previous-version)
  - [Install a package from a different release (Fedora Rawhide, etc.)](#install-a-package-from-a-different-release-fedora-rawhide-etc)
  - [Lock a package to its current version](#lock-a-package-to-its-current-version)
- [Tips and Best Practices](#tips-and-best-practices)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---
## What is DNF?

DNF replaced the older YUM (Yellowdog Updater Modified) package manager starting with Fedora 22. It is faster, uses less memory, and has a cleaner API. Under the hood, DNF:

- Resolves package **dependencies** automatically
- Downloads packages from **repositories** (remote or local collections of software)
- Verifies **GPG signatures** to ensure package integrity
- Maintains a **transaction history** so you can undo changes

---


[↑ Goto TOC](#table-of-contents)

## Basic Concepts

**Package** — A compressed archive (`.rpm` file) containing software, metadata, and installation instructions.

**Repository (repo)** — A server or directory hosting a collection of packages with an index DNF can read. Repos are defined in `/etc/yum.repos.d/`.

**Dependency** — A package that another package requires to function. DNF resolves these for you automatically.

**Transaction** — A set of operations (installs, updates, removals) applied together. If any step fails, the whole transaction is rolled back.

**Epoch, Version, Release (EVR)** — The three-part version string used to compare packages: `epoch:version-release.arch` (e.g., `2:5.14.0-284.fc38.x86_64`).

---


[↑ Goto TOC](#table-of-contents)

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


[↑ Goto TOC](#table-of-contents)

## Installing Packages


[↑ Goto TOC](#table-of-contents)

### Install a single package

```bash
sudo dnf install <package-name>
```

Example:

```bash
sudo dnf install vim
```

DNF will show you a summary of what will be installed (including dependencies) and ask for confirmation. Type `y` and press Enter to proceed.


[↑ Goto TOC](#table-of-contents)

### Install multiple packages at once

```bash
sudo dnf install vim git curl wget
```


[↑ Goto TOC](#table-of-contents)

### Install a local `.rpm` file

```bash
sudo dnf install ./mypackage.rpm
```

Using `dnf install` instead of `rpm -i` for local files ensures dependencies are resolved from your repos.


[↑ Goto TOC](#table-of-contents)

### Install a specific version

```bash
sudo dnf install vim-9.0.1234
```


[↑ Goto TOC](#table-of-contents)

### Reinstall a package

Useful if package files are corrupted:

```bash
sudo dnf reinstall vim
```

---


[↑ Goto TOC](#table-of-contents)

## Removing Packages


[↑ Goto TOC](#table-of-contents)

### Remove a single package

```bash
sudo dnf remove <package-name>
```

Example:

```bash
sudo dnf remove vim
```


[↑ Goto TOC](#table-of-contents)

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


[↑ Goto TOC](#table-of-contents)

## Updating Packages


[↑ Goto TOC](#table-of-contents)

### Update all packages

```bash
sudo dnf update
```

This fetches the latest package metadata and upgrades all installed packages with available updates.


[↑ Goto TOC](#table-of-contents)

### Update a specific package

```bash
sudo dnf update vim
```


[↑ Goto TOC](#table-of-contents)

### Check for available updates without installing

```bash
dnf check-update
```


[↑ Goto TOC](#table-of-contents)

### Upgrade vs Update

`dnf upgrade` and `dnf update` are functionally identical in modern DNF. Both handle version upgrades and obsoletes.


[↑ Goto TOC](#table-of-contents)

### Upgrade your entire distribution (e.g., Fedora release upgrade)

```bash
sudo dnf system-upgrade download --releasever=<version>
sudo dnf system-upgrade reboot
```

---


[↑ Goto TOC](#table-of-contents)

## Searching for Packages


[↑ Goto TOC](#table-of-contents)

### Search by name or summary

```bash
dnf search <keyword>
```

Example:

```bash
dnf search video editor
```


[↑ Goto TOC](#table-of-contents)

### Search in package names only

```bash
dnf search --name <keyword>
```


[↑ Goto TOC](#table-of-contents)

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


[↑ Goto TOC](#table-of-contents)

## Getting Package Information


[↑ Goto TOC](#table-of-contents)

### Show detailed package info

```bash
dnf info <package-name>
```

Displays version, description, size, repo, license, and more.


[↑ Goto TOC](#table-of-contents)

### List all installed packages

```bash
dnf list installed
```


[↑ Goto TOC](#table-of-contents)

### List all available packages

```bash
dnf list available
```


[↑ Goto TOC](#table-of-contents)

### List packages with available updates

```bash
dnf list updates
```


[↑ Goto TOC](#table-of-contents)

### Check if a specific package is installed

```bash
dnf list installed vim
```


[↑ Goto TOC](#table-of-contents)

### List files installed by a package

```bash
dnf repoquery -l <package-name>
```


[↑ Goto TOC](#table-of-contents)

### Show dependencies of a package

```bash
dnf repoquery --requires <package-name>
```


[↑ Goto TOC](#table-of-contents)

### Show what packages depend on a given package

```bash
dnf repoquery --whatrequires <package-name>
```

---


[↑ Goto TOC](#table-of-contents)

## Managing Repositories


[↑ Goto TOC](#table-of-contents)

### List all enabled repositories

```bash
dnf repolist
```


[↑ Goto TOC](#table-of-contents)

### List all repositories (enabled and disabled)

```bash
dnf repolist all
```


[↑ Goto TOC](#table-of-contents)

### Enable a repository temporarily (for one command)

```bash
sudo dnf install <package> --enablerepo=<repo-id>
```


[↑ Goto TOC](#table-of-contents)

### Enable a repository permanently

```bash
sudo dnf config-manager --enable <repo-id>
```


[↑ Goto TOC](#table-of-contents)

### Disable a repository permanently

```bash
sudo dnf config-manager --disable <repo-id>
```


[↑ Goto TOC](#table-of-contents)

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


[↑ Goto TOC](#table-of-contents)

### Enable Third-Party Repos (Fedora)

Popular third-party repos include **RPM Fusion**, which provides software not included in Fedora by default (e.g., media codecs, proprietary drivers):

```bash
sudo dnf install \
  https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

---


[↑ Goto TOC](#table-of-contents)

## Working with Package Groups

Groups are collections of related packages bundled together for convenience.


[↑ Goto TOC](#table-of-contents)

### List all available groups

```bash
dnf group list
```


[↑ Goto TOC](#table-of-contents)

### Get info about a group

```bash
dnf group info "Development Tools"
```


[↑ Goto TOC](#table-of-contents)

### Install a group

```bash
sudo dnf group install "Development Tools"
```


[↑ Goto TOC](#table-of-contents)

### Remove a group

```bash
sudo dnf group remove "Development Tools"
```


[↑ Goto TOC](#table-of-contents)

### Update all packages in a group

```bash
sudo dnf group update "Development Tools"
```

---


[↑ Goto TOC](#table-of-contents)

## History and Rollback

DNF keeps a log of all transactions, allowing you to view and undo changes.


[↑ Goto TOC](#table-of-contents)

### View transaction history

```bash
dnf history
```


[↑ Goto TOC](#table-of-contents)

### View details of a specific transaction

```bash
dnf history info <transaction-id>
```


[↑ Goto TOC](#table-of-contents)

### Undo a specific transaction

```bash
sudo dnf history undo <transaction-id>
```


[↑ Goto TOC](#table-of-contents)

### Redo a specific transaction

```bash
sudo dnf history redo <transaction-id>
```


[↑ Goto TOC](#table-of-contents)

### Roll back to a specific transaction

```bash
sudo dnf history rollback <transaction-id>
```

This reverts your system to the state it was in after the given transaction ID.

---


[↑ Goto TOC](#table-of-contents)

## Cleaning Up

Over time, DNF caches metadata and downloaded packages. Cleaning up can free disk space.


[↑ Goto TOC](#table-of-contents)

### Remove cached package data

```bash
sudo dnf clean packages
```


[↑ Goto TOC](#table-of-contents)

### Remove cached metadata

```bash
sudo dnf clean metadata
```


[↑ Goto TOC](#table-of-contents)

### Remove all cached data

```bash
sudo dnf clean all
```


[↑ Goto TOC](#table-of-contents)

### Remove orphaned packages (auto-installed, no longer needed)

```bash
sudo dnf autoremove
```


[↑ Goto TOC](#table-of-contents)

### Rebuild the metadata cache

```bash
sudo dnf makecache
```

---


[↑ Goto TOC](#table-of-contents)

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


[↑ Goto TOC](#table-of-contents)

### Example: Non-interactive update

```bash
sudo dnf update -y
```


[↑ Goto TOC](#table-of-contents)

### Example: Download a package without installing it

```bash
sudo dnf install --downloadonly --downloaddir=/tmp vim
```

---


[↑ Goto TOC](#table-of-contents)

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


[↑ Goto TOC](#table-of-contents)

## Common Recipes


[↑ Goto TOC](#table-of-contents)

### Install development tools

```bash
sudo dnf group install "Development Tools"
sudo dnf install gcc make cmake python3-devel
```


[↑ Goto TOC](#table-of-contents)

### Find and install a missing library

```bash
dnf provides libssl.so.3
sudo dnf install openssl-libs
```


[↑ Goto TOC](#table-of-contents)

### Keep your system up to date automatically (Fedora)

```bash
sudo dnf install dnf-automatic
sudo systemctl enable --now dnf-automatic.timer
```

Edit `/etc/dnf/automatic.conf` to configure behavior (e.g., download only vs. auto-apply updates).


[↑ Goto TOC](#table-of-contents)

### Downgrade a package to a previous version

```bash
sudo dnf downgrade vim
```


[↑ Goto TOC](#table-of-contents)

### Install a package from a different release (Fedora Rawhide, etc.)

```bash
sudo dnf install --releasever=rawhide <package>
```


[↑ Goto TOC](#table-of-contents)

### Lock a package to its current version

```bash
sudo dnf install python3-dnf-plugin-versionlock
sudo dnf versionlock add vim
```

---


[↑ Goto TOC](#table-of-contents)

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


[↑ Goto TOC](#table-of-contents)

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

[↑ Goto TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
