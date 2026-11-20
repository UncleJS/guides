# DNF Package Manager: A Beginner's Guide

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)


DNF (Dandified YUM) is the default package manager for RPM-based Linux distributions such as Fedora, RHEL (Red Hat Enterprise Linux), CentOS Stream, and AlmaLinux. It handles installing, updating, removing, and querying software packages, along with automatically resolving dependencies.


## Table of Contents

- [What is DNF?](#what-is-dnf)
- [dnf4 vs dnf5](#dnf4-vs-dnf5)
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


[↑ Back to TOC](#table-of-contents)

## dnf4 vs dnf5

Two generations of DNF are in use today, and both are run with the `dnf` command:

| Distribution | DNF generation | Check with |
|--------------|----------------|------------|
| Fedora 41 and newer | **dnf5** (`dnf` is a symlink to `dnf5`) | `dnf --version` shows `dnf5 version 5.x` |
| RHEL 8, 9, 10, CentOS Stream, AlmaLinux, Rocky Linux | **dnf4** (classic DNF) | `dnf --version` shows `4.x` |

Most everyday commands (`install`, `remove`, `upgrade`, `search`, `info`, `provides`, `history`, `group install`) are the same in both. The main differences this guide points out:

| Task | dnf5 (Fedora 41+) | dnf4 (RHEL 8/9/10) |
|------|-------------------|--------------------|
| Help for a command | `dnf <command> --help` | `dnf help <command>` or `dnf <command> --help` |
| Enable/disable a repo permanently | `dnf config-manager setopt <repo>.enabled=1` | `dnf config-manager --enable <repo>` (needs `dnf-plugins-core`) |
| Add a repo from a `.repo` URL | `dnf config-manager addrepo --from-repofile=<url>` | `dnf config-manager --add-repo <url>` |
| Download a package to a directory | `dnf download --destdir=<dir> <pkg>` | `dnf download --destdir=<dir> <pkg>` or `dnf install --downloadonly --downloaddir=<dir> <pkg>` |
| Version locking | built in: `dnf versionlock` | needs `python3-dnf-plugin-versionlock` |
| Release upgrade | built in: `dnf system-upgrade` | Fedora only, needs `dnf-plugin-system-upgrade` (RHEL uses Leapp) |
| Search names only | `dnf search --name <keyword>` | not available; use `dnf list '*keyword*'` |

Where a command differs, the sections below show it with a `# dnf5:` / `# dnf4:` comment.

---


[↑ Back to TOC](#table-of-contents)

## Basic Concepts

**Package** — A compressed archive (`.rpm` file) containing software, metadata, and installation instructions.

**Repository (repo)** — A server or directory hosting a collection of packages with an index DNF can read. Repos are defined in `/etc/yum.repos.d/`.

**Dependency** — A package that another package requires to function. DNF resolves these for you automatically.

**Transaction** — A set of operations (installs, updates, removals) applied together. If any step fails, the whole transaction is rolled back.

**Epoch, Version, Release (EVR)** — The three-part version string used to compare packages: `epoch:version-release` (e.g., `2:9.1.1000-1.fc41`). The epoch is often `0` and then omitted.

**NEVRA** — The full package identifier: `name-epoch:version-release.arch` (e.g., `vim-enhanced-2:9.1.1000-1.fc41.x86_64`).

---


[↑ Back to TOC](#table-of-contents)

## Getting Started

DNF is pre-installed on supported distributions. Most operations require root/sudo privileges.

Check that DNF is available and see its version:

```bash
dnf --version
```

Display the built-in help:

```bash
dnf --help
dnf <command> --help   # help for a specific command (dnf4 and dnf5)
dnf help <command>     # dnf4 only
```

---


[↑ Back to TOC](#table-of-contents)

## Installing Packages


[↑ Back to TOC](#table-of-contents)

### Install a single package

```bash
sudo dnf install <package-name>
```

Example:

```bash
sudo dnf install vim
```

DNF will show you a summary of what will be installed (including dependencies) and ask for confirmation. Type `y` and press Enter to proceed.


[↑ Back to TOC](#table-of-contents)

### Install multiple packages at once

```bash
sudo dnf install vim git curl wget
```


[↑ Back to TOC](#table-of-contents)

### Install a local `.rpm` file

```bash
sudo dnf install ./mypackage.rpm
```

Using `dnf install` instead of `rpm -i` for local files ensures dependencies are resolved from your repos.


[↑ Back to TOC](#table-of-contents)

### Install a specific version

Append the version (and optionally the release) to the package name. List the versions your repos offer first:

```bash
dnf list --showduplicates vim-enhanced
sudo dnf install vim-enhanced-9.1.1000-1.fc41
```

The epoch can also be given (`vim-enhanced-2:9.1.1000-1.fc41`). Exact versions vary by release; copy one from the `--showduplicates` output.


[↑ Back to TOC](#table-of-contents)

### Reinstall a package

Useful if package files are corrupted:

```bash
sudo dnf reinstall vim
```

---


[↑ Back to TOC](#table-of-contents)

## Removing Packages


[↑ Back to TOC](#table-of-contents)

### Remove a single package

```bash
sudo dnf remove <package-name>
```

Example:

```bash
sudo dnf remove vim
```


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

## Updating Packages


[↑ Back to TOC](#table-of-contents)

### Update all packages

```bash
sudo dnf update
```

This fetches the latest package metadata and upgrades all installed packages with available updates.


[↑ Back to TOC](#table-of-contents)

### Update a specific package

```bash
sudo dnf update vim
```


[↑ Back to TOC](#table-of-contents)

### Check for available updates without installing

```bash
dnf check-update
```


[↑ Back to TOC](#table-of-contents)

### Upgrade vs Update

`dnf upgrade` and `dnf update` are functionally identical in modern DNF. Both handle version upgrades and obsoletes.


[↑ Back to TOC](#table-of-contents)

### Upgrade your entire distribution (e.g., Fedora release upgrade)

This applies to **Fedora only**. RHEL major-version upgrades (for example 9 to 10) use **Leapp**, not `dnf system-upgrade`.

First bring the current release fully up to date, then download and reboot into the upgrade:

```bash
sudo dnf upgrade --refresh
# dnf4 only (Fedora 40 and older): sudo dnf install dnf-plugin-system-upgrade
sudo dnf system-upgrade download --releasever=<version>
sudo dnf system-upgrade reboot
```

---


[↑ Back to TOC](#table-of-contents)

## Searching for Packages


[↑ Back to TOC](#table-of-contents)

### Search by name or summary

```bash
dnf search <keyword>
```

Example:

```bash
dnf search video editor
```


[↑ Back to TOC](#table-of-contents)

### Search in package names only

```bash
# dnf5:
dnf search --name <keyword>
# dnf4 (no --name option): match package names with a glob
dnf list '*<keyword>*'
```


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

## Getting Package Information


[↑ Back to TOC](#table-of-contents)

### Show detailed package info

```bash
dnf info <package-name>
```

Displays version, description, size, repo, license, and more.


[↑ Back to TOC](#table-of-contents)

### List all installed packages

```bash
dnf list --installed
```


[↑ Back to TOC](#table-of-contents)

### List all available packages

```bash
dnf list available
```


[↑ Back to TOC](#table-of-contents)

### List packages with available updates

```bash
dnf list --upgrades
```


[↑ Back to TOC](#table-of-contents)

### Check if a specific package is installed

```bash
dnf list --installed vim
```


[↑ Back to TOC](#table-of-contents)

### List files installed by a package

```bash
dnf repoquery --installed --files <package-name>
```

`dnf repoquery` queries the **repositories** by default. Add `--installed` to query what is actually on your system (without it, you see the files of the repo version, even if the package is not installed). `rpm -ql <package-name>` gives the same result for installed packages.


[↑ Back to TOC](#table-of-contents)

### Show dependencies of a package

```bash
dnf repoquery --requires <package-name>              # from the repos
dnf repoquery --installed --requires <package-name>  # installed version
```


[↑ Back to TOC](#table-of-contents)

### Show what packages depend on a given package

```bash
dnf repoquery --whatrequires <package-name>              # packages in the repos
dnf repoquery --installed --whatrequires <package-name>  # installed packages only
```

---


[↑ Back to TOC](#table-of-contents)

## Managing Repositories


[↑ Back to TOC](#table-of-contents)

### List all enabled repositories

```bash
dnf repolist
```


[↑ Back to TOC](#table-of-contents)

### List all repositories (enabled and disabled)

```bash
dnf repolist --all
```


[↑ Back to TOC](#table-of-contents)

### Enable a repository temporarily (for one command)

```bash
sudo dnf install <package> --enablerepo=<repo-id>
```


[↑ Back to TOC](#table-of-contents)

### Enable a repository permanently

```bash
# dnf5:
sudo dnf config-manager setopt <repo-id>.enabled=1
# dnf4 (needs dnf-plugins-core):
sudo dnf config-manager --enable <repo-id>
```

Recent dnf5 releases also accept `sudo dnf config-manager enable <repo-id>`.


[↑ Back to TOC](#table-of-contents)

### Disable a repository permanently

```bash
# dnf5:
sudo dnf config-manager setopt <repo-id>.enabled=0
# dnf4 (needs dnf-plugins-core):
sudo dnf config-manager --disable <repo-id>
```


[↑ Back to TOC](#table-of-contents)

### Add a new repository

```bash
# dnf5:
sudo dnf config-manager addrepo --from-repofile=<url-of-.repo-file>
# dnf4 (needs dnf-plugins-core):
sudo dnf config-manager --add-repo <url-of-.repo-file>
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


[↑ Back to TOC](#table-of-contents)

### Enable Third-Party Repos (Fedora)

Popular third-party repos include **RPM Fusion**, which provides software not included in Fedora by default (e.g., media codecs, proprietary drivers):

```bash
sudo dnf install \
  https://download1.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm \
  https://download1.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
```

---


[↑ Back to TOC](#table-of-contents)

## Working with Package Groups

Groups are collections of related packages bundled together for convenience.


[↑ Back to TOC](#table-of-contents)

### List all available groups

```bash
dnf group list
```


[↑ Back to TOC](#table-of-contents)

### Get info about a group

```bash
dnf group info "Development Tools"
```


[↑ Back to TOC](#table-of-contents)

### Install a group

```bash
sudo dnf group install "Development Tools"
```


[↑ Back to TOC](#table-of-contents)

### Remove a group

```bash
sudo dnf group remove "Development Tools"
```


[↑ Back to TOC](#table-of-contents)

### Update all packages in a group

```bash
sudo dnf group upgrade "Development Tools"
```

---


[↑ Back to TOC](#table-of-contents)

## History and Rollback

DNF keeps a log of all transactions, allowing you to view and undo changes.


[↑ Back to TOC](#table-of-contents)

### View transaction history

```bash
dnf history
```


[↑ Back to TOC](#table-of-contents)

### View details of a specific transaction

```bash
dnf history info <transaction-id>
```


[↑ Back to TOC](#table-of-contents)

### Undo a specific transaction

```bash
sudo dnf history undo <transaction-id>
```


[↑ Back to TOC](#table-of-contents)

### Redo a specific transaction

```bash
sudo dnf history redo <transaction-id>
```


[↑ Back to TOC](#table-of-contents)

### Roll back to a specific transaction

```bash
sudo dnf history rollback <transaction-id>
```

This reverts your system to the state it was in after the given transaction ID.

---


[↑ Back to TOC](#table-of-contents)

## Cleaning Up

Over time, DNF caches metadata and downloaded packages. Cleaning up can free disk space.


[↑ Back to TOC](#table-of-contents)

### Remove cached package data

```bash
sudo dnf clean packages
```


[↑ Back to TOC](#table-of-contents)

### Remove cached metadata

```bash
sudo dnf clean metadata
```


[↑ Back to TOC](#table-of-contents)

### Remove all cached data

```bash
sudo dnf clean all
```


[↑ Back to TOC](#table-of-contents)

### Remove orphaned packages (auto-installed, no longer needed)

```bash
sudo dnf autoremove
```


[↑ Back to TOC](#table-of-contents)

### Rebuild the metadata cache

```bash
sudo dnf makecache
```

---


[↑ Back to TOC](#table-of-contents)

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
| `--allowerasing` | Allow removing installed packages to resolve conflicts (review the summary) |
| `--downloadonly` | Download packages without installing |
| `--downloaddir=<path>` | dnf4: directory for `--downloadonly` (dnf5: use `dnf download --destdir=<path>`) |
| `--setopt=<key=value>` | Override a config option temporarily |


[↑ Back to TOC](#table-of-contents)

### Example: Non-interactive update

```bash
sudo dnf update -y
```


[↑ Back to TOC](#table-of-contents)

### Example: Download a package without installing it

`dnf download` fetches the `.rpm` into a directory you choose and needs no `sudo`:

```bash
dnf download vim-enhanced --destdir=/tmp
```

`dnf download` is built into dnf5; on dnf4 it comes from `dnf-plugins-core`. dnf4 also supports `sudo dnf install --downloadonly --downloaddir=/tmp vim-enhanced`.

---


[↑ Back to TOC](#table-of-contents)

## Configuration

DNF's main configuration file is `/etc/dnf/dnf.conf`. Here are some useful settings:

```ini
[main]
# Always verify GPG signatures
gpgcheck=1
# Keep only the 3 most recent kernel versions
installonly_limit=3
# Fail if a repo is unavailable
skip_if_unavailable=False
# Download up to 5 packages simultaneously
max_parallel_downloads=5
```

Put comments on their own lines: DNF does not support `#` comments after a value on the same line.

Notes:

- `best=True` and `clean_requirements_on_remove=True` are already the defaults, so you don't need to set them.
- `fastestmirror=True` is often discouraged: the mirror system already picks a nearby mirror, and the latency test can make DNF slower.

You can also override any setting temporarily on the command line:

```bash
sudo dnf update --setopt=max_parallel_downloads=10
```

---


[↑ Back to TOC](#table-of-contents)

## Common Recipes


[↑ Back to TOC](#table-of-contents)

### Install development tools

```bash
sudo dnf group install "Development Tools"
sudo dnf install gcc make cmake python3-devel
```


[↑ Back to TOC](#table-of-contents)

### Find and install a missing library

```bash
dnf provides libssl.so.3
sudo dnf install openssl-libs
```


[↑ Back to TOC](#table-of-contents)

### Keep your system up to date automatically (Fedora)

```bash
sudo dnf install dnf-automatic
sudo systemctl enable --now dnf-automatic.timer
```

Edit `/etc/dnf/automatic.conf` to configure behavior (e.g., download only vs. auto-apply updates).


[↑ Back to TOC](#table-of-contents)

### Downgrade a package to a previous version

```bash
sudo dnf downgrade vim
```


[↑ Back to TOC](#table-of-contents)

### Install a package from a different release (Fedora Rawhide, etc.)

```bash
sudo dnf install --releasever=rawhide <package>
```

> ⚠️ **Warning:** This can pull in Rawhide versions of libraries and other dependencies, leaving your system with a mix of releases that may break later upgrades. Prefer a backport, a COPR, Flatpak or a container (for example `toolbox`) when possible, and review the transaction carefully.


[↑ Back to TOC](#table-of-contents)

### Lock a package to its current version

```bash
# dnf5: versionlock is built in
sudo dnf versionlock add vim-enhanced
# dnf4: install the plugin first
sudo dnf install python3-dnf-plugin-versionlock
sudo dnf versionlock add vim-enhanced
```

List locks with `dnf versionlock list` and remove one with `sudo dnf versionlock delete vim-enhanced`.

---


[↑ Back to TOC](#table-of-contents)

## Tips and Best Practices

**Always review the transaction summary.** Before typing `y`, read what DNF plans to install, remove, or update. Surprises are rare but possible.

**Use `dnf check-update` before updating.** It lets you see what's pending without committing to anything.

**Keep your cache fresh.** Run `sudo dnf makecache` periodically so package searches and installs are faster.

**Prefer `dnf install` over `rpm -i`.** Even for local `.rpm` files, DNF resolves dependencies from your repos automatically.

**Audit history after problems.** If something broke after an update, `dnf history` and `dnf history undo` are your friends.

**Use `--best --allowerasing` to resolve stubborn conflicts.** This tells DNF it can replace or remove conflicting packages to find a solution:

```bash
sudo dnf update --best --allowerasing
```

> ⚠️ **Caution:** `--allowerasing` may **remove** installed packages. Read the "Removing" part of the transaction summary before you confirm.

**Check enabled repos before troubleshooting.** A missing package is often in a disabled or un-added repo. Run `dnf repolist` to check.

**Be cautious with `--nogpgcheck`.** It disables a critical security check. Only use it for trusted local packages during testing.

---


[↑ Back to TOC](#table-of-contents)

## Quick Reference Cheat Sheet

```
dnf install <pkg>          Install a package
dnf remove <pkg>           Remove a package
dnf update                 Update all packages
dnf update <pkg>           Update a specific package
dnf search <keyword>       Search for packages
dnf info <pkg>             Show package details
dnf provides <file>        Find which package owns a file
dnf list --installed       List installed packages
dnf list --upgrades        List available updates
dnf repolist               Show enabled repos
dnf history                Show transaction history
dnf history undo <id>      Undo a transaction
dnf autoremove             Remove orphaned packages
dnf clean all              Clear all cached data
dnf group list             List package groups
dnf group install <group>  Install a package group
```

---

*This guide covers dnf5 (Fedora 41 and newer) and dnf4 (RHEL 8, 9 and 10, CentOS Stream, AlmaLinux and Rocky Linux). See [dnf4 vs dnf5](#dnf4-vs-dnf5) for the commands that differ.*

[↑ Back to TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
