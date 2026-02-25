---
Created:
  - "2026-02-19 06:51"
Tags: 
Work: 
Private: 
Tech: 
Words:
---




Access Control Lists (ACLs) in Linux provide a more flexible and granular way to manage file and directory permissions compared to the traditional user/group/others (ugo) model. While standard permissions are limited to three categories, ACLs allow you to assign specific permissions to multiple users or groups individually. This is particularly useful in shared environments, like servers or team projects, where you need fine-tuned access without creating numerous groups.

This guide builds on basic Linux permissions knowledge. We'll cover ACL concepts, setup, commands, examples, and advanced topics. Examples assume a terminal on a common distro like Ubuntu or Fedora. Some commands may require `sudo` for system files.


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


[↑ Goto TOC](#table-of-contents)

### Basics
- **Access Control Lists (ACLs)**: An extension to the standard permission system. Each file or directory can have an ACL that lists additional entries for users and groups beyond the basic owner, group, and others.
- **Types of ACLs**:
  - **Access ACLs**: Control immediate access to the file or directory itself.
  - **Default ACLs**: Applied to directories; they define permissions for new files and subdirectories created inside.


[↑ Goto TOC](#table-of-contents)

### Limitations of Standard Permissions
- Standard ugo permissions only allow one owner, one group, and a catch-all "others."
- If you need to grant access to multiple specific users (e.g., Alice can read/write, Bob can only read), you'd need complex group setups or risky "others" permissions.
- ACLs solve this by adding entries like: "user:alice:rwx" or "group:developers:r--".


[↑ Goto TOC](#table-of-contents)

### When to Use ACLs
- Shared folders with varied access needs.
- Environments with many users (e.g., web servers, databases).
- When standard permissions feel too restrictive or overly permissive.

ACLs are POSIX-compliant and supported on most modern filesystems like ext4, XFS, and Btrfs.


[↑ Goto TOC](#table-of-contents)

## 2. Enabling ACL Support

ACLs aren't always enabled by default. Check and enable them:


[↑ Goto TOC](#table-of-contents)

### Check Filesystem Support
- View mounted filesystems: `mount | grep acl` or `df -T`.
- If "acl" isn't in the options, you may need to remount or edit `/etc/fstab`.


[↑ Goto TOC](#table-of-contents)

### Install ACL Tools
- On Debian/Ubuntu: `sudo apt update && sudo apt install acl`
- On Fedora/RHEL: `sudo dnf install acl`
- On Arch: `sudo pacman -S acl`


[↑ Goto TOC](#table-of-contents)

### Enable on Filesystem
- For ext4 (common root filesystem):
  - Edit `/etc/fstab`: Add `acl` to the options column (e.g., `UUID=... / ext4 defaults,acl 0 1`).
  - Remount: `sudo mount -o remount,acl /`
- For new partitions: Use `mkfs.ext4 -O acl /dev/sdX`
- Verify: `tune2fs -l /dev/sdX | grep acl` (should show "Default mount options: user_xattr acl").

Without enabling, ACL commands will error out.


[↑ Goto TOC](#table-of-contents)

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


[↑ Goto TOC](#table-of-contents)

## 4. Setting ACLs with `setfacl`

The `setfacl` command modifies ACLs. Syntax:
```bash
setfacl [options] mode entry file
```

- **mode**: `-m` (modify/add), `-x` (remove), `-b` (remove all extended ACLs).
- **entry**: Format like `u:username:perms`, `g:groupname:perms`, `m:perms` (mask), `o:perms` (others), `d:...` (default ACL).
- **perms**: `rwx` or `-` (like standard, or octal like 6 for rw-).
- Options: `-R` (recursive), `--restore=file` (from backup).


[↑ Goto TOC](#table-of-contents)

### Adding/Modifying Entries
- Give Alice read/write: `setfacl -m u:alice:rw- myfile.txt`
- Give developers group execute: `setfacl -m g:developers:x mydirectory`
- Set for multiple: `setfacl -m u:alice:rw-,u:bob:r-- myfile.txt`


[↑ Goto TOC](#table-of-contents)

### Removing Entries
- Remove Alice's access: `setfacl -x u:alice myfile.txt`
- Remove all extended ACLs: `setfacl -b myfile.txt`


[↑ Goto TOC](#table-of-contents)

### Default ACLs for Directories
- Set default for new files: `setfacl -m d:u:alice:rw- mydirectory`
- New files inside will inherit these as access ACLs.
- View defaults: They appear prefixed with `default:` in `getfacl`.


[↑ Goto TOC](#table-of-contents)

### Recursive Application
- Apply to directory and contents: `setfacl -R -m u:alice:r-- sharedfolder`


[↑ Goto TOC](#table-of-contents)

## 5. The ACL Mask and Effective Permissions

- **Mask**: A special entry (`mask::`) that limits the maximum permissions for all named users/groups (not owner or others).
- Why? Prevents accidental over-permissioning.
- Effective permissions = Requested perms AND mask.
- In output, `#effective:` shows the result (e.g., if mask is `rw-` and entry is `rwx`, effective is `rw-`).
- Set mask: `setfacl -m m::rx myfile.txt`
- Automatic: When you add named entries, mask is set to union of group and named perms if not specified.
- To maximize: Set mask to `rwx` (but use cautiously).

Example:
- If group is `r--` and mask is `r--`, a `u:alice:rwx` becomes effective `r--`.


[↑ Goto TOC](#table-of-contents)

## 6. Inheritance and How ACLs Work with Standard Permissions

- **Inheritance**:
  - Default ACLs on directories propagate to new subfiles/directories.
  - For subdirectories: They get the default as both access and default ACLs.
  - For files: Only as access ACLs.
- **Interaction with Standard Perms**:
  - ACLs supplement, not replace, ugo.
  - `chmod` affects the base (user/group/other) and can adjust the mask indirectly.
  - If ACLs are present, `ls -l` shows a `+` at the end of permissions (e.g., `-rw-r--r--+`).
- **Order of Evaluation**:
  1. If user is owner: Use owner perms.
  2. If named user entry: Use that (masked).
  3. If in named group(s): Use the most permissive (masked).
  4. If in owning group: Use group perms.
  5. Else: Use others perms.


[↑ Goto TOC](#table-of-contents)

## 7. Examples


[↑ Goto TOC](#table-of-contents)

### Scenario 1: Shared Project File
- Create file: `touch project.txt`
- Standard: Owner (you) rw-, group r--, others --- (assuming umask 0022).
- Add ACL for Alice (rw) and Bob (r): `setfacl -m u:alice:rw,u:bob:r project.txt`
- Set mask to rw (to allow Alice's w): `setfacl -m m:rw project.txt`
- View: `getfacl project.txt`


[↑ Goto TOC](#table-of-contents)

### Scenario 2: Directory with Defaults
- Create dir: `mkdir teamfolder`
- Set default ACL for developers group (rx): `setfacl -m d:g:developers:rx teamfolder`
- New file inside: `touch teamfolder/newfile.txt`
- `getfacl teamfolder/newfile.txt` shows inherited group:developers:rx.


[↑ Goto TOC](#table-of-contents)

### Scenario 3: Recursive Secure Share
- `setfacl -R -m u:guest:r-- publicdir`
- Removes write access for guest in the whole tree.


[↑ Goto TOC](#table-of-contents)

## 8. Backup and Restore ACLs

- Backup: `getfacl -R sharedfolder > acl_backup.txt`
- Restore: `setfacl --restore=acl_backup.txt`
- Tools like `rsync` with `-A` preserve ACLs: `rsync -aA source/ dest/`


[↑ Goto TOC](#table-of-contents)

## 9. Advanced Topics

- **Filesystem Quotas and ACLs**: ACLs don't affect quotas; those are per-user/group.
- **SELinux/AppArmor**: ACLs work alongside mandatory access controls.
- **NFS/Samba Sharing**: ACL support varies; enable `acl` in exports or smb.conf.
- **Performance**: Minimal overhead, but many ACLs on large directories can slow listings.
- **Limitations**: Not all filesystems/apps support ACLs fully (e.g., FAT32 doesn't).
- **Debugging**: Use `strace setfacl` for errors. Common issues: Filesystem not mounted with ACLs.


[↑ Goto TOC](#table-of-contents)

## 10. Best Practices and Security

- **Start Simple**: Use ACLs only when ugo isn't enough.
- **Minimal Privileges**: Grant least access needed.
- **Monitor**: Regularly `getfacl` important files.
- **Avoid Conflicts**: Don't mix with `chmod 777`; use ACLs for precision.
- **Test**: Create test users/groups: `sudo useradd testuser; sudo passwd testuser; su - testuser` to verify access.
- **Security Risks**: Misconfigured ACLs can expose data—audit periodically.
- **Alternatives**: For complex setups, consider tools like `rbac` or filesystem features like Btrfs subvolumes.

ACLs empower precise control but add complexity. Practice in a non-critical directory. For more, check `man getfacl` and `man setfacl`. If you have specific scenarios, ask!
[↑ Goto TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
