# Detailed Guide to Linux Access Control Lists (ACLs)

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

Access Control Lists (ACLs) in Linux provide a more flexible and granular way to manage file and directory permissions compared to the traditional user/group/others (ugo) model. While standard permissions are limited to three categories, ACLs allow you to assign specific permissions to multiple users or groups individually. This is particularly useful in shared environments, like servers or team projects, where you need fine-tuned access without creating numerous groups.

This guide builds on basic Linux permissions knowledge. We'll cover ACL concepts, setup, commands, examples, and advanced topics. Examples assume a terminal on Fedora or RHEL (other distributions work the same way). Some commands may require `sudo` for system files.


## Table of Contents

- [1. What Are ACLs and Why Use Them?](#1-what-are-acls-and-why-use-them)
  - [Basics](#basics)
  - [Limitations of Standard Permissions](#limitations-of-standard-permissions)
  - [When to Use ACLs](#when-to-use-acls)
- [2. Enabling ACL Support](#2-enabling-acl-support)
  - [Check Filesystem Support](#check-filesystem-support)
  - [Install ACL Tools](#install-acl-tools)
  - [Enable on Filesystem](#enable-on-filesystem)
- [3. Viewing ACLs with `getfacl`](#3-viewing-acls-with-getfacl)
- [4. Setting ACLs with `setfacl`](#4-setting-acls-with-setfacl)
  - [Adding/Modifying Entries](#addingmodifying-entries)
  - [Removing Entries](#removing-entries)
  - [Default ACLs for Directories](#default-acls-for-directories)
  - [Recursive Application](#recursive-application)
- [5. The ACL Mask and Effective Permissions](#5-the-acl-mask-and-effective-permissions)
- [6. Inheritance and How ACLs Work with Standard Permissions](#6-inheritance-and-how-acls-work-with-standard-permissions)
- [7. Examples](#7-examples)
  - [Scenario 1: Shared Project File](#scenario-1-shared-project-file)
  - [Scenario 2: Directory with Defaults](#scenario-2-directory-with-defaults)
  - [Scenario 3: Recursive Secure Share](#scenario-3-recursive-secure-share)
- [8. Backup and Restore ACLs](#8-backup-and-restore-acls)
- [9. Advanced Topics](#9-advanced-topics)
- [10. Best Practices and Security](#10-best-practices-and-security)

---
## 1. What Are ACLs and Why Use Them?


[↑ Back to TOC](#table-of-contents)

### Basics
- **Access Control Lists (ACLs)**: An extension to the standard permission system. Each file or directory can have an ACL that lists additional entries for users and groups beyond the basic owner, group, and others.
- **Types of ACLs**:
  - **Access ACLs**: Control immediate access to the file or directory itself.
  - **Default ACLs**: Applied to directories; they define permissions for new files and subdirectories created inside.


[↑ Back to TOC](#table-of-contents)

### Limitations of Standard Permissions
- Standard ugo permissions only allow one owner, one group, and a catch-all "others."
- If you need to grant access to multiple specific users (e.g., Alice can read/write, Bob can only read), you'd need complex group setups or risky "others" permissions.
- ACLs solve this by adding entries like: "user:alice:rwx" or "group:developers:r--".


[↑ Back to TOC](#table-of-contents)

### When to Use ACLs
- Shared folders with varied access needs.
- Environments with many users (e.g., web servers, databases).
- When standard permissions feel too restrictive or overly permissive.

ACLs are POSIX-compliant and supported on most modern filesystems like ext4, XFS, and Btrfs.


[↑ Back to TOC](#table-of-contents)

## 2. Enabling ACL Support

On Fedora and RHEL, ACLs work out of the box: XFS (the RHEL default) always supports them, and ext4 and Btrfs enable them by default. You normally only need to check:


[↑ Back to TOC](#table-of-contents)

### Check Filesystem Support
- Find the filesystem type: `df -T /path` or `findmnt -T /path`.
- XFS and Btrfs: ACLs are always available; there is nothing to enable.
- ext4: `acl` is a default mount option, so it usually does **not** appear in `mount` output. Check the superblock defaults with `sudo tune2fs -l /dev/sdX | grep "Default mount options"` (should list `acl`).
- The quickest test: `touch testfile && setfacl -m u:$USER:r testfile`. An "Operation not supported" error means the filesystem lacks ACL support.


[↑ Back to TOC](#table-of-contents)

### Install ACL Tools
- On Fedora/RHEL: `sudo dnf install acl` (usually already installed)
- On Debian/Ubuntu: `sudo apt update && sudo apt install acl`
- On Arch: `sudo pacman -S acl`


[↑ Back to TOC](#table-of-contents)

### Enable on Filesystem
`acl` is a **mount option**, not a filesystem feature, so there is no `mkfs` flag for it. Only if an ext4 filesystem was mounted with `noacl` (or its default mount options were changed):
- Edit `/etc/fstab`: Add `acl` to the options column (e.g., `UUID=... /data ext4 defaults,acl 0 2`).
- Remount: `sudo mount -o remount,acl /data`
- Or set it as a superblock default: `sudo tune2fs -o acl /dev/sdX`
- Verify: `sudo tune2fs -l /dev/sdX | grep "Default mount options"` (should show `user_xattr acl`).

Without ACL support, `setfacl` fails with "Operation not supported".


[↑ Back to TOC](#table-of-contents)

## 3. Viewing ACLs with `getfacl`

Use `getfacl` to display ACLs:

```bash
getfacl filename
```

Example output for a file with ACLs:
```
# file: myfile.txt
# owner: user
# group: group
user::rw-
user:alice:rwx          # effective:rw-
group::r--
group:developers:r-x    # effective:r--
mask::rw-
other::r--
```

Breakdown:
- **# lines**: Comments with file name, owner, group.
- **user::**: Standard user (owner) permissions.
- **user:username:perms**: Additional user-specific ACLs.
- **group::**: Standard group permissions.
- **group:groupname:perms**: Additional group-specific ACLs.
- **mask::**: The "effective" mask (explained later).
- **other::**: Standard others permissions.

Options:
- `-R`: Recursive (for directories).
- `--omit-header`: Cleaner output without # lines.
- If no extra ACLs, it shows just standard permissions.


[↑ Back to TOC](#table-of-contents)

## 4. Setting ACLs with `setfacl`

The `setfacl` command modifies ACLs. Syntax:
```bash
setfacl [options] mode entry file
```

- **mode**: `-m` (modify/add), `-x` (remove), `-b` (remove all extended ACLs).
- **entry**: Format like `u:username:perms`, `g:groupname:perms`, `m:perms` (mask), `o:perms` (others), `d:...` (default ACL).
- **perms**: `rwx` or `-` (like standard, or octal like 6 for rw-). A capital `X` means execute only for directories and for files that already have execute for someone; use it when applying recursively.
- Options: `-R` (recursive), `--restore=file` (from backup).


[↑ Back to TOC](#table-of-contents)

### Adding/Modifying Entries
- Give Alice read/write: `setfacl -m u:alice:rw- myfile.txt`
- Give developers group execute: `setfacl -m g:developers:x mydirectory`
- Set for multiple: `setfacl -m u:alice:rw-,u:bob:r-- myfile.txt`


[↑ Back to TOC](#table-of-contents)

### Removing Entries
- Remove Alice's access: `setfacl -x u:alice myfile.txt`
- Remove all extended ACLs: `setfacl -b myfile.txt`


[↑ Back to TOC](#table-of-contents)

### Default ACLs for Directories
- Set default for new files and subdirectories: `setfacl -m d:u:alice:rwX mydirectory`
- New files inside inherit these as access ACLs. New subdirectories inherit them as both access and default ACLs.
- Default ACLs only affect **new** items; existing contents are unchanged.
- Directories need execute (search/traverse) to be entered. Include `x`/`X` in default entries so new subdirectories are usable; new regular files still don't get execute unless the creating program asks for it (the creation mode, e.g. 0666 for `touch`, limits the inherited permissions).
- The user also needs an access ACL (at least `x`) on the directory itself, and on every parent directory, to reach the files: `setfacl -m u:alice:rwx mydirectory`.
- View defaults: They appear prefixed with `default:` in `getfacl`.


[↑ Back to TOC](#table-of-contents)

### Recursive Application
- Apply to directory and contents: `setfacl -R -m u:alice:rX sharedfolder`
- Use `rX`, not `r--`: with plain `r` Alice can list directory names but cannot enter subdirectories or open files in them. `X` adds execute to directories (and already-executable files) only.
- `-R` changes existing items only; add a default ACL as well for files created later: `setfacl -R -m d:u:alice:rX sharedfolder`


[↑ Back to TOC](#table-of-contents)

## 5. The ACL Mask and Effective Permissions

- **Mask**: A special entry (`mask::`) that sets the upper limit for the **group class**: all named users, all named groups, **and the owning group** (`group::`). It does not affect the owner (`user::`) or others (`other::`).
- Why? It lets you cap everything granted through ACLs in one place.
- Effective permissions = entry perms AND mask.
- In output, `#effective:` shows the result (e.g., if mask is `rw-` and entry is `rwx`, effective is `rw-`).
- Set mask: `setfacl -m m::rx myfile.txt`
- Automatic: Unless you set the mask explicitly (or use `-n`), `setfacl` recalculates it as the union of the owning group and all named user and group entries.
- **chmod and the mask**: On a file with an ACL, `ls -l` shows the **mask** in the group position, and `chmod g=...` changes the mask, not the `group::` entry. For example, `chmod g-w file` removes write from every named user and group as well. A later `setfacl -m` recalculates the mask again.

Example:
- If the mask is `r--`, a `u:alice:rwx` entry is effective `r--`, and a `group::rw-` entry is effective `r--` too.


[↑ Back to TOC](#table-of-contents)

## 6. Inheritance and How ACLs Work with Standard Permissions

- **Inheritance**:
  - Default ACLs on directories propagate to new subfiles/directories.
  - For subdirectories: They get the default as both access and default ACLs.
  - For files: Only as access ACLs.
- **Interaction with Standard Perms**:
  - ACLs supplement, not replace, ugo.
  - `chmod` changes the owner and other entries directly. When an ACL has a mask, the group bits of `chmod` set the mask instead of `group::` (see section 5).
  - If ACLs are present, `ls -l` shows a `+` at the end of permissions (e.g., `-rw-r--r--+`).
- **Order of Evaluation**:
  1. If user is owner: Use owner perms (`user::`), no mask.
  2. If a named user entry matches: Use that, limited by the mask.
  3. If the user is in the owning group or any named group: access is granted if **any** matching group entry (limited by the mask) contains the requested perms; otherwise denied.
  4. Else: Use others perms (`other::`), no mask.

  The first matching step decides; for example, an owner with `user::---` is denied even if a group entry would allow access.


[↑ Back to TOC](#table-of-contents)

## 7. Examples


[↑ Back to TOC](#table-of-contents)

### Scenario 1: Shared Project File
- Create file: `touch project.txt`
- Standard with umask 0022: Owner (you) rw-, group r--, others r-- (`-rw-r--r--`). Remove others' access first: `chmod o= project.txt`
- Add ACL for Alice (rw) and Bob (r): `setfacl -m u:alice:rw,u:bob:r project.txt`
- The mask is recalculated to `rw-` automatically. Set it by hand only if it was reduced later (e.g., by `chmod g-w`): `setfacl -m m::rw project.txt`
- Alice and Bob also need execute on the parent directories to reach the file.
- View: `getfacl project.txt`


[↑ Back to TOC](#table-of-contents)

### Scenario 2: Directory with Defaults
- Create dir: `mkdir teamfolder`
- Let developers enter the folder now: `setfacl -m g:developers:rx teamfolder`
- Set default ACL for developers group (rx): `setfacl -m d:g:developers:rx teamfolder`
- New file inside: `touch teamfolder/newfile.txt`
- `getfacl teamfolder/newfile.txt` shows inherited `group:developers:r-x  #effective:r--`: `touch` creates files with mode 0666, so the mask has no `x`. New subdirectories (`mkdir`) get the full `r-x`.


[↑ Back to TOC](#table-of-contents)

### Scenario 3: Recursive Secure Share
- `setfacl -R -m u:guest:rX publicdir`
- Gives guest read access to the whole tree: read on files, read and enter (`x`) on directories. It does **not** remove access guest already has through group or other permissions.
- For new files too: `setfacl -R -m d:u:guest:rX publicdir`
- To make guest's access strictly read-only, also check that guest is not in the owning group and that `other::` has no `w`.


[↑ Back to TOC](#table-of-contents)

## 8. Backup and Restore ACLs

- Backup: `getfacl -R sharedfolder > acl_backup.txt`
- Restore: `setfacl --restore=acl_backup.txt`
- Tools like `rsync` with `-A` preserve ACLs: `rsync -aA source/ dest/`


[↑ Back to TOC](#table-of-contents)

## 9. Advanced Topics

- **Filesystem Quotas and ACLs**: ACLs don't affect quotas; those are per-user/group.
- **SELinux/AppArmor**: ACLs work alongside mandatory access controls.
- **NFS/Samba Sharing**: NFSv4 (the default on Fedora/RHEL) uses its own NFSv4 ACL model, managed with `nfs4_getfacl`/`nfs4_setfacl` from `nfs4-acl-tools`; the server maps them to POSIX ACLs on the exported filesystem. NFSv3 passes POSIX ACLs through the NFSACL protocol, so `getfacl`/`setfacl` work. Samba maps POSIX ACLs to Windows ACLs (or stores Windows ACLs with `vfs objects = acl_xattr`).
- **Performance**: Minimal overhead, but many ACLs on large directories can slow listings.
- **Limitations**: Not all filesystems/apps support ACLs fully (e.g., FAT32 doesn't).
- **Debugging**: Use `strace setfacl` for errors. Common issues: Filesystem not mounted with ACLs.


[↑ Back to TOC](#table-of-contents)

## 10. Best Practices and Security

- **Start Simple**: Use ACLs only when ugo isn't enough.
- **Minimal Privileges**: Grant least access needed.
- **Monitor**: Regularly `getfacl` important files.
- **Avoid Conflicts**: Don't mix with `chmod 777`; use ACLs for precision.
- **Test**: Create test users/groups: `sudo useradd testuser; sudo passwd testuser; su - testuser` to verify access.
- **Security Risks**: Misconfigured ACLs can expose data—audit periodically.
- **Alternatives**: For simple cases, a dedicated group plus a setgid directory (`chmod 2770`) is often easier to manage than many ACL entries. SELinux adds mandatory controls on top of ACLs.

ACLs empower precise control but add complexity. Practice in a non-critical directory. For more, check `man getfacl` and `man setfacl`. If you have specific scenarios, ask!

[↑ Back to TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
