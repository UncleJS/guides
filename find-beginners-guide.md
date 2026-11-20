# A Beginner's Guide to the `find` Command

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)


The `find` command is one of the most powerful tools in Unix/Linux. It lets you search for files and directories based on almost any criteria imaginable — name, size, type, date modified, permissions, and more — and then act on the results. This guide walks you through everything you need to get started.


## Table of Contents

- [Basic Syntax](#basic-syntax)
- [Searching by Name](#searching-by-name)
- [Searching by Type](#searching-by-type)
- [Searching by Size](#searching-by-size)
- [Searching by Time](#searching-by-time)
- [Searching by Permissions](#searching-by-permissions)
- [Searching by Owner](#searching-by-owner)
- [Limiting Search Depth](#limiting-search-depth)
- [Combining Conditions](#combining-conditions)
  - [AND (both must be true)](#and-both-must-be-true)
  - [OR (either must be true)](#or-either-must-be-true)
  - [NOT (negate a condition)](#not-negate-a-condition)
  - [Grouping with Parentheses](#grouping-with-parentheses)
- [Acting on Results](#acting-on-results)
  - [`-print` (default)](#-print-default)
  - [`-delete`](#-delete)
  - [`-exec`](#-exec)
  - [`-exec` in Batches (more efficient)](#-exec-in-batches-more-efficient)
  - [Piping to `xargs`](#piping-to-xargs)
  - [`-ls`](#-ls)
- [Practical Examples](#practical-examples)
- [Tips and Gotchas](#tips-and-gotchas)
- [Quick Reference Card](#quick-reference-card)

---
## Basic Syntax

```
find [path] [options] [expression]
```

- **path** — Where to start the search. Use `.` for the current directory, `/` for the entire filesystem, or any specific path like `/home/user/documents`.
- **options** — Modify how `find` behaves (e.g., depth limits).
- **expression** — The criteria to filter by (name, size, type, etc.).

If you omit the path, GNU `find` (Fedora/RHEL, Debian/Ubuntu) defaults to the current directory. BSD/macOS `find` requires a path, so write `find .` to be portable.

```bash
# Find everything in the current directory and below
find .

# Find everything starting from the root (the whole system)
find /
```

> **Note:** Searching from `/` can return thousands of results and take a long time. It may also produce "Permission denied" errors for directories you don't have access to. You can suppress those with `2>/dev/null`.

```bash
find / 2>/dev/null
```

---


[↑ Back to TOC](#table-of-contents)

## Searching by Name

The `-name` flag matches filenames. It is **case-sensitive** by default.

```bash
# Find a file called "report.txt" anywhere in the current directory
find . -name "report.txt"

# Use a wildcard to find all .log files
find . -name "*.log"

# Case-insensitive search with -iname
find . -iname "readme.md"
```

Wildcards follow standard shell globbing:

| Pattern | Meaning |
|---|---|
| `*` | Any number of characters |
| `?` | Exactly one character |
| `[abc]` | One character from the set |

```bash
# Find files starting with "backup" followed by any characters
find . -name "backup*"

# Find files with a single-character extension
find . -name "*.?"
```

---


[↑ Back to TOC](#table-of-contents)

## Searching by Type

The `-type` flag filters by the kind of filesystem entry.

| Flag | Meaning |
|---|---|
| `-type f` | Regular file |
| `-type d` | Directory |
| `-type l` | Symbolic link |
| `-type p` | Named pipe |
| `-type s` | Socket |

```bash
# Find only directories named "logs"
find . -type d -name "logs"

# Find only regular files (not directories or links)
find . -type f

# Find all symbolic links
find . -type l
```

---


[↑ Back to TOC](#table-of-contents)

## Searching by Size

The `-size` flag matches files by size. The format is a number followed by a unit:

| Unit | Meaning |
|---|---|
| `c` | Bytes |
| `k` | Kibibytes (1024 bytes) |
| `M` | Mebibytes (1024 × 1024 bytes) |
| `G` | Gibibytes (1024 × 1024 × 1024 bytes) |

A number with no unit counts 512-byte blocks.

You can also use `+` (greater than) and `-` (less than) prefixes:

```bash
# Files that take up 100 KiB when rounded up (more than 99 KiB, up to 100 KiB)
find . -size 100k

# Files larger than 50 MiB
find . -size +50M

# Files smaller than 1 KiB (use bytes: -1k only matches empty files, see below)
find . -size -1024c

# Files larger than 1 MiB and at most 99 MiB
find . -size +1M -size -100M
```

> **Gotcha: sizes are rounded up.** `find` rounds each file's size **up** to whole units before comparing. A 1-byte file counts as `1k`, so `-size -1k` ("less than 1 unit") only matches **empty** files, and `-size -100M` means 99 MiB or less. When you need precision, use bytes (`c`).

---


[↑ Back to TOC](#table-of-contents)

## Searching by Time

`find` offers three time-based criteria. Each counts in **days** by default but can use minutes with the `min` variants.

| Flag | Meaning |
|---|---|
| `-mtime n` | Modified `n` days ago (see rounding below) |
| `-atime n` | Last accessed `n` days ago |
| `-ctime n` | Status changed `n` days ago |
| `-mmin n` | Modified `n` minutes ago |
| `-amin n` | Accessed `n` minutes ago |
| `-cmin n` | Status changed `n` minutes ago |

Again, `+` means "more than" and `-` means "less than".

> **Gotcha: days are whole 24-hour periods.** For `-mtime`, `-atime` and `-ctime`, `find` divides the file's age by 24 hours and **discards the fractional part**. A file modified 47 hours ago has an age of `1`, so:
>
> - `-mtime 1` matches files modified 24 to just under 48 hours ago
> - `-mtime +1` means an age **greater than 1**, i.e. at least **2 full days** (48 hours or more)
> - `-mtime -1` means an age of `0`, i.e. modified in the last 24 hours
>
> Also note that Fedora/RHEL and most Linux systems mount filesystems with `relatime`, so access times (`-atime`) are only updated about once a day and are not exact.

```bash
# Files modified in the last 7 days (less than 7 × 24 hours ago)
find . -mtime -7

# Files not accessed for more than 30 full days (31 days or more)
find . -atime +30

# Files modified more than 60 minutes ago
find . -mmin +60

# Files modified in the last 10 minutes
find . -mmin -10
```

You can also compare against a reference file using `-newer`:

```bash
# Files newer than reference.txt
find . -newer reference.txt
```

---


[↑ Back to TOC](#table-of-contents)

## Searching by Permissions

The `-perm` flag matches files by their permission bits. The prefix changes how the mode is compared:

| Form | Matches when |
|---|---|
| `-perm 644` | The permissions are **exactly** `644` |
| `-perm -644` | **All** of these bits are set (other bits may also be set) |
| `-perm /644` | **Any** of these bits is set |

```bash
# Find files with exact permissions 644
find . -perm 644

# Files with the executable bit set for anyone (owner, group OR others): "any" match
find . -perm /111

# Files executable by owner, group AND others: "all" match
find . -perm -111

# Files readable by owner (this bit is set, whatever the others are)
find . -perm -400

# Find world-writable files (a security concern)
find / -type f -perm -o+w 2>/dev/null

# Find SUID files (common security audit)
find / -perm /4000 2>/dev/null
```

---


[↑ Back to TOC](#table-of-contents)

## Searching by Owner

```bash
# Find files owned by a specific user
find . -user alice

# Find files owned by a specific group
find . -group developers

# Find files owned by a specific UID
find . -uid 1001
```

---


[↑ Back to TOC](#table-of-contents)

## Limiting Search Depth

By default, `find` recurses through all subdirectories. You can limit this with `-maxdepth` and `-mindepth`.

```bash
# Only search in the current directory (no subdirectories)
find . -maxdepth 1 -name "*.txt"

# Search only 2 levels deep
find . -maxdepth 2 -type f

# Skip the top level and start at depth 2
find . -mindepth 2 -maxdepth 3 -name "*.conf"
```

---


[↑ Back to TOC](#table-of-contents)

## Combining Conditions

By default, multiple conditions are joined with an implicit **AND** — all conditions must be true. You can be explicit or use other logical operators.


[↑ Back to TOC](#table-of-contents)

### AND (both must be true)

```bash
# Files that are .sh AND larger than 10k
find . -name "*.sh" -size +10k

# Explicit AND with -and
find . -name "*.sh" -and -size +10k
```


[↑ Back to TOC](#table-of-contents)

### OR (either must be true)

```bash
# Find .jpg or .png files
find . -name "*.jpg" -or -name "*.png"
```


[↑ Back to TOC](#table-of-contents)

### NOT (negate a condition)

```bash
# Find everything that is NOT a .log file
find . -not -name "*.log"

# Shorthand: !
find . ! -name "*.log"
```


[↑ Back to TOC](#table-of-contents)

### Grouping with Parentheses

Use escaped parentheses `\( \)` to group conditions:

```bash
# Files that are (.jpg or .png) AND larger than 1MB
find . \( -name "*.jpg" -or -name "*.png" \) -size +1M
```

---


[↑ Back to TOC](#table-of-contents)

## Acting on Results

The real power of `find` comes from being able to **do something** with the files it finds.


[↑ Back to TOC](#table-of-contents)

### `-print` (default)

Prints the path of each result. This is what `find` does when you don't specify an action.

```bash
find . -name "*.txt" -print
```


[↑ Back to TOC](#table-of-contents)

### `-delete`

Deletes the matched files. **Use with caution — there is no confirmation prompt.**

```bash
# Step 1: preview what would be deleted
find . -name "*.tmp" -print

# Step 2: same command, with -print replaced by -delete
find . -name "*.tmp" -delete
```

> **Danger: `-delete` must come last.** `find` evaluates the expression left to right, and `-delete` is an action, not a filter. If you put it **before** your tests, it runs on every file it reaches before the tests are checked:
>
> ```bash
> # WRONG: deletes EVERYTHING under the current directory
> find . -delete -name "*.tmp"
> ```
>
> Always run the command with `-print` first to preview what will be deleted, then swap `-print` for `-delete` at the **end** of the command.
>
> `-delete` also turns on `-depth` automatically (directory contents are processed before the directory itself), so it cannot be combined with `-prune`, and it only removes directories that are empty.


[↑ Back to TOC](#table-of-contents)

### `-exec`

Runs a command on each result. The `{}` is a placeholder for the found file, and `\;` terminates the command.

```bash
# Print detailed info about each .log file
find . -name "*.log" -exec ls -lh {} \;

# Change permissions on all .sh files
find . -name "*.sh" -exec chmod +x {} \;

# Copy all .conf files to a backup directory
find /etc -name "*.conf" -exec cp {} /backup/ \;
```


[↑ Back to TOC](#table-of-contents)

### `-exec` in Batches (more efficient)

Using `+` instead of `\;` passes all results to the command at once (like `xargs`), which is faster:

```bash
find . -name "*.log" -exec rm {} +
```


[↑ Back to TOC](#table-of-contents)

### Piping to `xargs`

An alternative to `-exec` that is often faster for large result sets:

```bash
find . -name "*.txt" | xargs grep "TODO"

# Use -print0 and -0 to safely handle filenames with spaces
find . -name "*.txt" -print0 | xargs -0 grep "TODO"
```


[↑ Back to TOC](#table-of-contents)

### `-ls`

A shortcut for running `ls -l` on each result:

```bash
find . -name "*.sh" -ls
```

---


[↑ Back to TOC](#table-of-contents)

## Practical Examples

Here are real-world scenarios where `find` shines.

```bash
# Find and delete all empty files
find . -type f -empty -delete

# Find all empty directories
find . -type d -empty

# Find the 10 largest files in /var
find /var -type f -printf "%s %p\n" | sort -rn | head -10

# Find all files modified in the last 24 hours
find . -mtime -1 -type f

# Find all .py files and count lines in each
find . -name "*.py" -exec wc -l {} +

# Find files with a specific string in their name (case-insensitive)
find . -iname "*invoice*"

# Find all broken symbolic links
find . -type l ! -exec test -e {} \; -print
# GNU find shortcut for the same thing
find . -xtype l

# Find duplicate-looking files by name
find . -name "*.bak" -type f

# Search only in specific directories and exclude others
find . -path ./node_modules -prune -o -name "*.js" -print

# Find all files changed in the last hour
find . -mmin -60 -type f
```

---


[↑ Back to TOC](#table-of-contents)

## Tips and Gotchas

**Always quote your patterns.** Without quotes, the shell may expand wildcards before `find` sees them, leading to unexpected results.

```bash
# Wrong — shell may expand * before find sees it
find . -name *.txt

# Right
find . -name "*.txt"
```

**Preview before deleting.** Always run your `find` command with `-print` instead of `-delete` or `-exec rm` first to see what matches, and keep `-delete` as the last item on the command line.

**Suppress permission errors.** Redirect stderr to `/dev/null` to hide "Permission denied" noise:

```bash
find / -name "*.conf" 2>/dev/null
```

**Handle spaces in filenames.** Use `-print0` with `xargs -0` when filenames might contain spaces or special characters.

**`-exec ... \;` vs `-exec ... +`**. The `\;` version runs the command once per file; the `+` version batches them. Use `+` when possible for better performance.

**`-prune` to skip directories.** To exclude a directory from the search:

```bash
# Search everything except node_modules
find . -path ./node_modules -prune -o -type f -print
```

**`find` vs `locate`.** `find` searches the live filesystem in real time; `locate` uses a pre-built index and is much faster but may be out of date. Use `find` when accuracy matters, `locate` when speed matters.

---


[↑ Back to TOC](#table-of-contents)

## Quick Reference Card

| Task | Command |
|---|---|
| Find by name | `find . -name "*.txt"` |
| Case-insensitive name | `find . -iname "readme*"` |
| Find directories | `find . -type d` |
| Find files over 100MB | `find . -size +100M` |
| Modified in last 7 days | `find . -mtime -7` |
| Find and delete | `find . -name "*.tmp" -delete` |
| Find and run a command | `find . -name "*.sh" -exec chmod +x {} \;` |
| Exclude a directory | `find . -path ./dir -prune -o -print` |
| Find empty files | `find . -type f -empty` |
| Limit search depth | `find . -maxdepth 2` |

---

With practice, `find` becomes an indispensable part of your daily workflow. Start with simple name searches and gradually layer in size, time, and permission filters as you grow more comfortable. Happy searching!

[↑ Back to TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
