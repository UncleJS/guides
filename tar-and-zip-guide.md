# Beginner's Guide to `tar` and `zip`

A practical introduction to archiving and compressing files on the command line.


## Table of Contents

- [What Are Archives and Compression?](#what-are-archives-and-compression)
- [tar](#tar)
  - [tar Basic Syntax](#tar-basic-syntax)
  - [Creating tar Archives](#creating-tar-archives)
  - [Extracting tar Archives](#extracting-tar-archives)
  - [Listing tar Contents](#listing-tar-contents)
  - [Compression with tar](#compression-with-tar)
  - [Common tar Examples](#common-tar-examples)
  - [tar Flags Cheat Sheet](#tar-flags-cheat-sheet)
- [zip](#zip)
  - [zip Basic Syntax](#zip-basic-syntax)
  - [Creating zip Archives](#creating-zip-archives)
  - [Extracting zip Archives](#extracting-zip-archives)
  - [Listing zip Contents](#listing-zip-contents)
  - [Common zip Examples](#common-zip-examples)
  - [zip Flags Cheat Sheet](#zip-flags-cheat-sheet)
- [tar vs. zip](#tar-vs-zip)
- [Tips, Tricks, and Gotchas](#tips-tricks-and-gotchas)

---
## What Are Archives and Compression?

Before diving in, it helps to understand two distinct concepts that often get bundled together:

**Archiving** means combining multiple files and directories into a single file. It preserves the structure (folder hierarchy, permissions, timestamps) but does not necessarily make anything smaller.

**Compression** means reducing the size of a file using a mathematical algorithm. Compression can be applied to a single file or to an archive.

`tar` was originally designed purely as an **archiving** tool and relies on separate programs (gzip, bzip2, xz) for compression. `zip` does **both** at the same time.

---


[↑ Goto TOC](#table-of-contents)

## tar

`tar` stands for **T**ape **AR**chive. Despite the old-fashioned name, it remains the standard archiving tool on Linux and macOS. Files it creates are called **tarballs** and typically end in `.tar`, `.tar.gz` (or `.tgz`), `.tar.bz2`, or `.tar.xz`.


[↑ Goto TOC](#table-of-contents)

### tar Basic Syntax

```
tar [OPTIONS] [ARCHIVE_NAME] [FILES/DIRECTORIES]
```

The first argument after `tar` is always one of the **main operation flags**:

| Flag | Long form | Meaning |
|------|-----------|---------|
| `-c` | `--create` | Create a new archive |
| `-x` | `--extract` | Extract files from an archive |
| `-t` | `--list` | List contents of an archive |
| `-r` | `--append` | Append files to an existing archive |
| `-u` | `--update` | Update files in an existing archive |

These are always combined with `-f`, which tells tar the name of the archive file.

---


[↑ Goto TOC](#table-of-contents)

### Creating tar Archives

**Create a basic (uncompressed) archive:**

```bash
tar -cf archive.tar file1.txt file2.txt folder/
```

This bundles `file1.txt`, `file2.txt`, and `folder/` into `archive.tar`.

**Verbose output (see what's being added):**

```bash
tar -cvf archive.tar folder/
```

The `-v` flag prints each filename as it's processed — useful for confirming things look right.

---


[↑ Goto TOC](#table-of-contents)

### Extracting tar Archives

**Extract everything into the current directory:**

```bash
tar -xf archive.tar
```

**Extract to a specific directory:**

```bash
tar -xf archive.tar -C /path/to/destination/
```

The `-C` flag changes the output directory. The destination must already exist.

**Extract a single file from an archive:**

```bash
tar -xf archive.tar path/inside/archive/file.txt
```

You need to specify the exact path as it appears inside the archive (check with `-t` first).

---


[↑ Goto TOC](#table-of-contents)

### Listing tar Contents

**See what's inside an archive without extracting:**

```bash
tar -tf archive.tar
```

**With details (permissions, owner, size, date):**

```bash
tar -tvf archive.tar
```

---


[↑ Goto TOC](#table-of-contents)

### Compression with tar

tar delegates compression to external tools. You enable them with a single flag:

| Flag | Algorithm | Extension | Speed | Compression |
|------|-----------|-----------|-------|-------------|
| `-z` | gzip | `.tar.gz` / `.tgz` | Fast | Moderate |
| `-j` | bzip2 | `.tar.bz2` | Slow | Better |
| `-J` | xz | `.tar.xz` | Very slow | Best |

**Create a gzip-compressed archive:**

```bash
tar -czf archive.tar.gz folder/
```

**Create a bzip2-compressed archive:**

```bash
tar -cjf archive.tar.bz2 folder/
```

**Create an xz-compressed archive:**

```bash
tar -cJf archive.tar.xz folder/
```

**Extracting compressed archives** works the same way — tar auto-detects the compression:

```bash
tar -xf archive.tar.gz
tar -xf archive.tar.bz2
tar -xf archive.tar.xz
```

You can also pass the compression flag explicitly (`-xzf`, `-xjf`, `-xJf`), but it's not required with modern tar.

---


[↑ Goto TOC](#table-of-contents)

### Common tar Examples

**Back up your home directory:**

```bash
tar -czf home_backup.tar.gz ~/
```

**Archive a project directory, excluding a folder:**

```bash
tar -czf project.tar.gz my_project/ --exclude='my_project/node_modules'
```

**Extract and strip the top-level directory:**

```bash
tar -xf archive.tar.gz --strip-components=1
```

This is useful when an archive wraps everything in a single folder and you just want the contents.

**Append files to an existing (uncompressed) archive:**

```bash
tar -rf archive.tar newfile.txt
```

Note: You cannot append to a compressed archive (`.tar.gz`, etc.).

**Create an archive and pipe it over SSH:**

```bash
tar -czf - folder/ | ssh user@remote "cat > archive.tar.gz"
```

Using `-f -` sends output to stdout instead of a file.

---


[↑ Goto TOC](#table-of-contents)

### tar Flags Cheat Sheet

| Flag | Meaning |
|------|---------|
| `-c` | Create archive |
| `-x` | Extract archive |
| `-t` | List contents |
| `-f FILE` | Specify archive filename (required) |
| `-v` | Verbose — print filenames |
| `-z` | Use gzip compression |
| `-j` | Use bzip2 compression |
| `-J` | Use xz compression |
| `-C DIR` | Extract to DIR |
| `-r` | Append files to archive |
| `--exclude=PATTERN` | Exclude matching files |
| `--strip-components=N` | Strip N leading path components on extract |
| `-p` | Preserve file permissions |

---


[↑ Goto TOC](#table-of-contents)

## zip

`zip` is the format most commonly associated with Windows, but it works on all platforms. Unlike `tar`, zip compresses each file individually before bundling, which means you can extract a single file from a zip without decompressing the whole thing. It's also natively supported by every major operating system with no extra software needed.


[↑ Goto TOC](#table-of-contents)

### zip Basic Syntax

```
zip [OPTIONS] ARCHIVE_NAME FILES/DIRECTORIES
```

Extraction uses a separate command:

```
unzip [OPTIONS] ARCHIVE_NAME
```

---


[↑ Goto TOC](#table-of-contents)

### Creating zip Archives

**Zip a single file:**

```bash
zip archive.zip file.txt
```

**Zip multiple files:**

```bash
zip archive.zip file1.txt file2.txt file3.txt
```

**Zip a directory (recursively):**

```bash
zip -r archive.zip folder/
```

The `-r` flag is essential — without it, zip only processes the folder itself, not its contents.

**Set compression level (0–9, default is 6):**

```bash
zip -9 -r archive.zip folder/   # Maximum compression
zip -0 -r archive.zip folder/   # No compression (just bundle)
```

**Create a password-protected archive:**

```bash
zip -e archive.zip file.txt
```

You'll be prompted to enter and confirm a password. Note that zip encryption is considered weak by modern standards — for sensitive data, use 7-zip with AES-256 or GPG.

**Exclude files matching a pattern:**

```bash
zip -r archive.zip folder/ --exclude "*.log"
```

---


[↑ Goto TOC](#table-of-contents)

### Extracting zip Archives

**Extract to the current directory:**

```bash
unzip archive.zip
```

**Extract to a specific directory:**

```bash
unzip archive.zip -d /path/to/destination/
```

**Extract a single file:**

```bash
unzip archive.zip file.txt
```

**Overwrite existing files without prompting:**

```bash
unzip -o archive.zip
```

**Never overwrite existing files:**

```bash
unzip -n archive.zip
```

**Extract a password-protected archive:**

```bash
unzip -P mypassword archive.zip
# or just run unzip archive.zip and enter the password when prompted
```

---


[↑ Goto TOC](#table-of-contents)

### Listing zip Contents

**List files in the archive:**

```bash
unzip -l archive.zip
```

**Verbose listing with more detail:**

```bash
unzip -v archive.zip
```

**Test the archive for errors:**

```bash
unzip -t archive.zip
```

---


[↑ Goto TOC](#table-of-contents)

### Common zip Examples

**Zip everything in a directory but exclude hidden files:**

```bash
zip -r archive.zip folder/ --exclude "*/.*"
```

**Update an existing zip with changed files only:**

```bash
zip -u archive.zip updated_file.txt
```

**Delete a file from an existing zip:**

```bash
zip -d archive.zip file_to_remove.txt
```

**Split a large archive into parts (e.g., 100MB each):**

```bash
zip -r -s 100m archive.zip large_folder/
```

---


[↑ Goto TOC](#table-of-contents)

### zip Flags Cheat Sheet

| Flag | Meaning |
|------|---------|
| `-r` | Recurse into directories |
| `-0` to `-9` | Compression level (0=none, 9=max) |
| `-e` | Encrypt with password |
| `-P PASSWORD` | Set password (insecure — use `-e` instead) |
| `-u` | Update (add/replace changed files only) |
| `-d` | Delete file from archive |
| `-s SIZE` | Split archive into parts |
| `--exclude PATTERN` | Exclude files matching pattern |
| `-j` | Junk (strip) directory paths |
| `-q` | Quiet mode |

**For `unzip`:**

| Flag | Meaning |
|------|---------|
| `-d DIR` | Extract to DIR |
| `-l` | List contents |
| `-v` | Verbose listing |
| `-t` | Test archive integrity |
| `-o` | Overwrite without prompting |
| `-n` | Never overwrite existing files |
| `-P PASSWORD` | Provide password |
| `-q` | Quiet mode |

---


[↑ Goto TOC](#table-of-contents)

## tar vs. zip

| Feature | tar (+ gzip/xz) | zip |
|---------|-----------------|-----|
| Platform | Linux/macOS native | Universal |
| Compression | External (gzip, bzip2, xz) | Built-in |
| Preserves permissions | Yes | Limited |
| Preserves symlinks | Yes | No |
| Single-file extraction | Harder | Easy |
| Streaming support | Yes | No |
| Password protection | No (use GPG) | Yes (weak) |
| Best for | Linux backups, source code, servers | Sharing with Windows users |

**Use `tar`** when working primarily on Linux/macOS, creating backups, or when you need to preserve file permissions and symlinks.

**Use `zip`** when sharing files with Windows users, when you need password protection for casual use, or when recipients need to extract individual files easily.

---


[↑ Goto TOC](#table-of-contents)

## Tips, Tricks, and Gotchas

**Always list before extracting.** Use `tar -tf archive.tar` or `unzip -l archive.zip` before extracting to confirm the archive doesn't dump files all over your current directory.

**The "tar bomb" problem.** Some archives don't have a top-level folder and will scatter their contents into your current directory. Extract into a new folder to be safe:

```bash
mkdir output && tar -xf archive.tar -C output/
```

**Compression choice matters for time vs. space.** For most purposes, gzip (`.tar.gz`) is the sweet spot. Only choose bzip2 or xz if file size is critical and you have time to wait.

**tar requires `-f` before the archive name.** Forgetting it is a very common mistake — tar will try to read from a tape device and hang.

**Dot files and hidden files.** Both tar and zip include hidden files (starting with `.`) by default. Keep this in mind when archiving home directories.

**Checking if commands are available.** On macOS, `zip` and `unzip` come pre-installed. On minimal Linux systems, you may need to install them:

```bash
sudo apt install zip unzip   # Debian/Ubuntu
sudo dnf install zip unzip   # Fedora/RHEL
```

`tar` is almost always pre-installed on any Unix-like system.

**Progress for large archives.** tar has no built-in progress bar, but you can pipe through `pv` (pipe viewer) if installed:

```bash
tar -czf - large_folder/ | pv > archive.tar.gz
```

---

*Happy archiving!*

[↑ Goto TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/)
