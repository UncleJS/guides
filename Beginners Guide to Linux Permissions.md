# Beginner’s Guide to Linux Permissions

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)

Welcome to this comprehensive beginner's guide to Linux permissions! Linux is a multi-user operating system, and permissions are a fundamental part of its security model. They control who can read, write, or execute files and directories. This guide will walk you through the basics, step by step, using simple explanations, examples, and commands. We'll assume you're using a terminal on a Linux distribution like Fedora or RHEL (other distributions work the same way).

By the end, you'll understand how to view, modify, and manage permissions effectively. Let's dive in!


## Table of Contents

- [1. What Are Linux Permissions?](#1-what-are-linux-permissions)
- [2. File Types in Linux](#2-file-types-in-linux)
- [3. Viewing Permissions](#3-viewing-permissions)
- [4. Understanding Permission Symbols](#4-understanding-permission-symbols)
- [5. Owners and Groups](#5-owners-and-groups)
- [6. Changing Permissions with `chmod`](#6-changing-permissions-with-chmod)
  - [Symbolic Mode](#symbolic-mode)
  - [Numeric (Octal) Mode](#numeric-octal-mode)
- [7. Changing Ownership with `chown` and `chgrp`](#7-changing-ownership-with-chown-and-chgrp)
- [8. Special Permissions](#8-special-permissions)
- [9. Umask: Default Permissions](#9-umask-default-permissions)
- [10. Access Control Lists (ACLs)](#10-access-control-lists-acls)
- [11. Common Scenarios and Best Practices](#11-common-scenarios-and-best-practices)
- [12. Tips for Beginners](#12-tips-for-beginners)

---
## 1. What Are Linux Permissions?

In Linux, every file and directory has associated permissions that define:
- **Who** can access it (users, groups, or others).
- **What** they can do (read, write, or execute).

Permissions prevent unauthorized access, ensuring system stability and security. For example:
- System files are often restricted to the root user.
- Your home directory is private by default on Fedora/RHEL (mode `700`, set by `HOME_MODE` in `/etc/login.defs`). Files you create elsewhere are **not** private: with the default umask `0022`, new files are `644` (readable by everyone) and directories `755`.

Permissions are stored in the file's metadata and can be viewed or changed using commands.


[↑ Back to TOC](#table-of-contents)

## 2. File Types in Linux

Before permissions, note that Linux treats everything as a file, but there are types indicated by the first character in listings:
- `-`: Regular file (e.g., text documents).
- `d`: Directory (folder).
- `l`: Symbolic link (shortcut to another file).
- `c` or `b`: Special files (e.g., devices like keyboards or hard drives).
- `s`: Socket (for inter-process communication).
- `p`: Named pipe (for data streaming).

You'll see this when using the `ls -l` command.


[↑ Back to TOC](#table-of-contents)

## 3. Viewing Permissions

Use the `ls` command with the `-l` flag to list files with details:

```bash
ls -l
```

Example output:
```
drwxr-xr-x  2 user group 4096 Feb 19 10:00 mydirectory
-rw-r--r--  1 user group 1024 Feb 19 09:00 myfile.txt
```

Breakdown:
- **First character**: File type (e.g., `d` for directory, `-` for file).
- **Next 9 characters**: Permissions (divided into 3 groups of 3: user, group, others).
- **Number**: Hard link count.
- **user**: Owner's username.
- **group**: Group's name.
- **4096/1024**: File size in bytes.
- **Date/Time**: Last modification.
- **Name**: File or directory name.

To view hidden files too, use `ls -la`.


[↑ Back to TOC](#table-of-contents)

## 4. Understanding Permission Symbols

Permissions are represented as three sets of `rwx` (read, write, execute) for:
- **User (u)**: The file owner.
- **Group (g)**: Users in the file's group.
- **Others (o)**: Everyone else.
- **All (a)**: Shortcut for u+g+o.

Symbols:
- `r`: Read permission (view file contents or list directory).
- `w`: Write permission (modify file or add/remove files in directory).
- `x`: Execute permission (run file as program or enter directory with `cd`).
- `-`: No permission.

In the example `-rw-r--r--`:
- User: `rw-` (read and write, no execute).
- Group: `r--` (read only).
- Others: `r--` (read only).

For directories, `x` allows traversal (entering the directory), even if you can't list contents without `r`.


[↑ Back to TOC](#table-of-contents)

## 5. Owners and Groups

- **Owner**: The user who created the file (or last changed ownership). Defaults to your username.
- **Group**: A collection of users. Files get the creator's primary group (or the directory's group if the directory has setgid). Fedora/RHEL use *user private groups*: each new user gets a personal primary group with the same name (e.g., user `alice`, group `alice`).

View your groups: `groups` or `id`.

Why groups? They allow shared access without giving everyone full rights. For example, a "developers" group can edit project files.


[↑ Back to TOC](#table-of-contents)

## 6. Changing Permissions with `chmod`

The `chmod` (change mode) command modifies permissions. It has two modes: symbolic and numeric (octal).


[↑ Back to TOC](#table-of-contents)

### Symbolic Mode
Use letters like `u`, `g`, `o`, `a` with `+` (add), `-` (remove), `=` (set exactly).

Examples:
- Add execute for user: `chmod u+x myfile.sh`
- Remove write for group and others: `chmod go-w myfile.txt`
- Set read/write for all: `chmod a=rw sharedfile.txt`
- Make directory readable/executable by everyone: `chmod a+rx mydirectory`

Apply recursively (for directories and contents): `chmod -R a+rx mydirectory`


[↑ Back to TOC](#table-of-contents)

### Numeric (Octal) Mode
Permissions as numbers (base 8):
- `r` = 4, `w` = 2, `x` = 1.
- Add them: e.g., `rwx` = 7 (4+2+1), `rw-` = 6 (4+2), `r--` = 4.

Three digits: user, group, others.

Examples:
- `chmod 644 myfile.txt` (user: rw-, group: r--, others: r--)
- `chmod 755 myscript.sh` (user: rwx, group: r-x, others: r-x)
- `chmod 700 privatefile` (user: rwx, everyone else: ---)

Octal is efficient for setting all at once.


[↑ Back to TOC](#table-of-contents)

## 7. Changing Ownership with `chown` and `chgrp`

- `chown` (change owner): Change owner and optionally group.
  - Syntax: `chown [newowner]:[newgroup] file`
  - Example: `chown alice:developers projectfile.txt`
  - Recursive: `chown -R alice:developers projectdir`

- `chgrp` (change group): Change group only.
  - Example: `chgrp developers projectfile.txt`

Only root can change a file's owner to another user, so `chown alice ...` needs `sudo` (even if you own the file). As the owner, you can change the group with `chgrp` (or `chown :group`) only to a group you are a member of; any other change needs `sudo`.


[↑ Back to TOC](#table-of-contents)

## 8. Special Permissions

Beyond basic rwx, there are special bits:

- **Setuid (s/S)**: For executables, runs as the file's owner (not the user running it). Appears as `s` in user execute position (e.g., `-rwsr-xr-x`). Use `chmod u+s`. Common for commands like `passwd`.
- **Setgid (s/S)**: Runs as the file's group. For directories, new files inherit the directory's group. `chmod g+s`.
- **Sticky Bit (t/T)**: For directories, prevents users from deleting others' files (e.g., /tmp). `chmod +t directory` or `chmod o+t`. Appears as `t` in others execute position (e.g., `drwxrwxrwt`).

In listings:
- Lowercase `s/t`: Permission + execute.
- Uppercase `S/T`: Permission without execute (rarely useful).

Set with octal: Add 4 (setuid), 2 (setgid), 1 (sticky) as a fourth digit (e.g., `chmod 4755` for setuid + 755).


[↑ Back to TOC](#table-of-contents)

## 9. Umask: Default Permissions

Umask sets default permissions for new files/directories. Programs request a mode (usually 666 for files and 777 for directories), and every bit that is set in the umask is **removed** (masked out) from that mode: `mode AND NOT umask`. It is a bitmask, not subtraction.

View umask: `umask` (e.g., `0022` removes write for group and others, so files default to 644 and directories to 755). `umask -S` shows the result symbolically.

Why "subtract" is wrong: with umask `0033`, files get `666 & ~033 = 644`, not `666 - 033 = 633`.

Change: `umask 0027` (stricter defaults: files 640, directories 750) or `umask 0077` (private: files 600, directories 700).

It's per-session; add it to `~/.bashrc` for permanence. The system default on Fedora/RHEL is `UMASK 022` in `/etc/login.defs`.


[↑ Back to TOC](#table-of-contents)

## 10. Access Control Lists (ACLs)

Basic permissions are limited to user/group/others. ACLs provide finer control, like per-user permissions.

- Check if supported: `getfacl file`
- Set: `setfacl -m u:alice:rwx file` (give alice rwx)
- View: `getfacl file`
- Remove: `setfacl -x u:alice file`

ACLs are advanced but useful for shared environments. The `acl` package is usually installed on Fedora/RHEL; if not, `sudo dnf install acl` (on Debian/Ubuntu: `sudo apt install acl`).


[↑ Back to TOC](#table-of-contents)

## 11. Common Scenarios and Best Practices

- **Secure Home Directory**: `chmod 700 ~/private` (only you can access).
- **Web Server Files**: Often `chmod 644` for files, `755` for directories, readable by the `apache` group on Fedora/RHEL (`www-data` on Debian/Ubuntu). Don't give the web server write access unless it needs it, and keep the SELinux label (`httpd_sys_content_t`) correct.
- **Scripts**: Make executable with `chmod +x script.sh`.
- **Shared Folders**: Use groups and setgid for inheritance.
- **Avoid 777**: Never use `chmod 777` unless necessary—it's insecure.
- **Use `sudo` Wisely**: For system files, but understand risks.
- **Backup First**: Before bulk changes (e.g., `chmod -R`), test on a copy.
- **Troubleshooting**: "Permission denied"? Check with `ls -l` and adjust.
- **Inheritance**: New files in directories inherit group if setgid is on.


[↑ Back to TOC](#table-of-contents)

## 12. Tips for Beginners

- Practice in a safe directory: `mkdir testdir; cd testdir; touch file1 file2`
- Man Pages: `man chmod`, `man chown` for details.
- GUI Tools: File managers like Nautilus have permission tabs, but CLI is more powerful.
- Security: Permissions are your first defense—keep them tight.
- Experiment: Create users/groups with `useradd`/`groupadd` to test.

If you encounter errors, search for them (e.g., "chmod permission denied"). This guide covers the essentials—happy Linuxing! If you have questions, feel free to ask.

[↑ Back to TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
