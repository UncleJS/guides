# A Beginner's Guide to rsync

## What is rsync?

`rsync` (Remote Sync) is a fast, versatile command-line tool for copying and synchronizing files and directories — either locally or between a local machine and a remote server. Unlike a plain `cp` command, rsync is smart: it only transfers the parts of files that have changed, making it extremely efficient for backups, deployments, and file management.

**Key advantages of rsync:**
- Only transfers changed data (delta transfer algorithm)
- Works locally and over a network (via SSH)
- Preserves file permissions, timestamps, symlinks, and ownership
- Supports compression during transfer
- Provides dry-run mode so you can preview changes before applying them
- Resumable transfers — great for large files over slow connections

---

## Installation

rsync comes pre-installed on most Linux distributions and macOS. To check if it's available:

```bash
rsync --version
```

If it's not installed, you can install it with your package manager:

```bash
# Debian / Ubuntu
sudo apt install rsync

# Fedora / RHEL / CentOS
sudo dnf install rsync

# macOS (via Homebrew)
brew install rsync

# Arch Linux
sudo pacman -S rsync
```

---

## Basic Syntax

```bash
rsync [OPTIONS] SOURCE DESTINATION
```

- **SOURCE** — the file or directory you want to copy *from*
- **DESTINATION** — where you want to copy *to*

Both source and destination can be local paths or remote paths (using `user@host:/path`).

---

## Your First rsync Commands

### Copy a single file locally

```bash
rsync file.txt /backup/file.txt
```

### Copy a directory locally

```bash
rsync -r /home/alice/documents/ /backup/documents/
```

The `-r` flag means **recursive** — it copies the directory and everything inside it.

> ⚠️ **Trailing slash gotcha:** This is one of the most common rsync pitfalls.
> - `rsync -r source/ destination/` → copies the *contents* of `source` into `destination`
> - `rsync -r source destination/` → copies the `source` *folder itself* into `destination`
>
> When in doubt, use a trailing slash on the source to avoid creating nested directories.

---

## Essential Flags

These are the options you'll use most often:

| Flag | Long Form | Description |
|------|-----------|-------------|
| `-r` | `--recursive` | Copy directories recursively |
| `-a` | `--archive` | Archive mode — preserves permissions, timestamps, symlinks, owner, group. Implies `-rlptgoD` |
| `-v` | `--verbose` | Show files being transferred |
| `-z` | `--compress` | Compress data during transfer (useful over slow networks) |
| `-h` | `--human-readable` | Print file sizes in human-readable format (KB, MB, GB) |
| `-n` | `--dry-run` | Simulate the transfer without actually doing anything |
| `-P` | `--progress --partial` | Show progress and allow resuming interrupted transfers |
| `--delete` | | Delete files in destination that no longer exist in source |
| `-e` | `--rsh=COMMAND` | Specify the remote shell to use (usually SSH) |
| `--exclude` | | Exclude files or directories matching a pattern |
| `--include` | | Include files matching a pattern (used alongside `--exclude`) |

---

## The Archive Flag (`-a`)

`-a` is the workhorse of rsync. It's equivalent to `-rlptgoD` and ensures files are copied faithfully:

- `-r` Recursive
- `-l` Copy symlinks as symlinks
- `-p` Preserve permissions
- `-t` Preserve timestamps
- `-g` Preserve group ownership
- `-o` Preserve owner (requires root)
- `-D` Preserve device and special files

**Most backups should use `-a`:**

```bash
rsync -av /home/alice/ /backup/alice/
```

---

## Dry Run (Always Recommended First)

Before running a destructive sync (especially with `--delete`), always preview what rsync will do:

```bash
rsync -avn --delete /home/alice/ /backup/alice/
```

The `-n` flag means nothing is actually changed. Review the output, then re-run without `-n` to apply.

---

## Syncing Over SSH (Remote Transfers)

rsync uses SSH by default for remote transfers, which means it's encrypted and secure.

### Local → Remote

```bash
rsync -avz /home/alice/documents/ user@192.168.1.10:/backup/documents/
```

### Remote → Local

```bash
rsync -avz user@192.168.1.10:/var/www/html/ /local/backup/html/
```

### Using a non-standard SSH port

```bash
rsync -avz -e "ssh -p 2222" /home/alice/ user@server.com:/backup/
```

### Using an SSH key file

```bash
rsync -avz -e "ssh -i ~/.ssh/mykey.pem" /home/alice/ user@server.com:/backup/
```

---

## Showing Progress

For large transfers, use `-P` to see a progress bar and transfer speed:

```bash
rsync -avhP /home/alice/ /backup/alice/
```

Output will look like:

```
documents/report.pdf
      4.20M 100%    2.34MB/s    0:00:01 (xfr#3, to-chk=42/87)
```

---

## Excluding Files and Directories

Use `--exclude` to skip certain files or patterns during a sync.

### Exclude a specific file

```bash
rsync -av --exclude='secret.txt' /home/alice/ /backup/alice/
```

### Exclude a directory

```bash
rsync -av --exclude='node_modules/' /home/alice/project/ /backup/project/
```

### Exclude multiple patterns

```bash
rsync -av \
  --exclude='*.log' \
  --exclude='.cache/' \
  --exclude='node_modules/' \
  /home/alice/project/ /backup/project/
```

### Use an exclude file

If you have many exclusions, put them in a file:

```
# exclude.txt
*.log
*.tmp
.cache/
node_modules/
.git/
```

Then reference it:

```bash
rsync -av --exclude-from='exclude.txt' /home/alice/project/ /backup/project/
```

---

## Keeping Destination in Sync with `--delete`

By default, rsync only adds or updates files — it never removes anything from the destination. If you want the destination to be a true mirror of the source (deleting files that no longer exist in the source), use `--delete`:

```bash
rsync -av --delete /home/alice/ /backup/alice/
```

> ⚠️ **Use `--delete` carefully.** Always do a dry run first (`-n`) so you don't accidentally wipe files you wanted to keep.

You can also use `--delete-dry-run` to see what would be deleted without actually doing it.

---

## Bandwidth Limiting

Throttle rsync so it doesn't saturate your network connection during large transfers:

```bash
# Limit to 1 MB/s
rsync -avz --bwlimit=1000 /home/alice/ user@server.com:/backup/
```

The value is in kilobytes per second.

---

## Practical Recipes

### Full local backup

```bash
rsync -avhP --delete /home/alice/ /mnt/external-drive/backup/alice/
```

### Backup to a remote server over SSH

```bash
rsync -avhzP --delete /var/www/html/ deploy@myserver.com:/var/www/html/
```

### Sync two directories, excluding hidden files

```bash
rsync -avh --exclude='.*' /home/alice/docs/ /backup/docs/
```

### Download a directory from a server

```bash
rsync -avhzP user@server.com:/home/alice/photos/ /local/photos/
```

### Copy only files newer than those in destination

```bash
rsync -avhu /home/alice/ /backup/alice/
```

The `-u` (`--update`) flag skips files that are newer in the destination.

### Mirror a website for offline viewing

```bash
rsync -avhz --delete user@mysite.com:/var/www/html/ ./site-mirror/
```

---

## Understanding rsync Output

When you run `rsync -av`, each transferred file is shown with a change summary. With `-i` (`--itemize-changes`), you get a detailed code like:

```
>f+++++++++ newfile.txt
.f..t...... updated.txt
*deleting   oldfile.txt
```

The format is `YXcstpoguax` where:
- `Y` — update type (`>` = file sent, `<` = file received, `*` = message, `.` = no update)
- `X` — file type (`f` = file, `d` = directory, `L` = symlink)
- The remaining letters indicate what attributes changed (checksum, size, timestamp, permissions, owner, group, etc.)

---

## Common Mistakes to Avoid

**1. Forgetting the trailing slash on source**
`rsync -a source destination/` vs `rsync -a source/ destination/` behave differently. Always verify with a dry run.

**2. Using `--delete` without a dry run**
You could accidentally delete important files. Always run `-n` first.

**3. Not using `-z` over slow connections**
Compression (`-z`) can dramatically speed up transfers over the internet, but may slow things down on fast local networks where CPU is the bottleneck.

**4. Syncing as root unnecessarily**
Running rsync as root can overwrite file ownership in unintended ways. Use the minimum required privileges.

**5. Confusing rsync with a backup tool**
rsync mirrors — it doesn't version files. If you accidentally delete a file in the source and run `rsync --delete`, it's gone from the destination too. For versioned backups, consider tools like `restic`, `borgbackup`, or `timeshift` built on top of rsync.

---

## Quick Reference Cheat Sheet

```bash
# Basic local copy
rsync -avh source/ destination/

# Dry run (preview changes)
rsync -avhn source/ destination/

# Mirror with deletions
rsync -avh --delete source/ destination/

# Remote push (local → server)
rsync -avhzP source/ user@host:/path/to/dest/

# Remote pull (server → local)
rsync -avhzP user@host:/path/to/source/ destination/

# Exclude patterns
rsync -avh --exclude='*.log' --exclude='node_modules/' source/ destination/

# Limit bandwidth (in KB/s)
rsync -avhz --bwlimit=2000 source/ user@host:/dest/

# Use a specific SSH key and port
rsync -avhz -e "ssh -i ~/.ssh/key.pem -p 2222" source/ user@host:/dest/
```

---

## Further Reading

- `man rsync` — the full manual page (very thorough)
- [rsync official website](https://rsync.samba.org/)
- [rsync algorithm paper](https://www.andrew.cmu.edu/course/15-749/READINGS/required/cas/tridgell96.pdf) — how the delta-transfer algorithm works under the hood

---

*Happy syncing!*
