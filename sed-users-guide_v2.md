# A User's Guide to `sed` — The Linux Stream Editor

## What is `sed`?

`sed` (Stream EDitor) is a powerful, non-interactive command-line tool for parsing and transforming text. It reads input line by line, applies editing commands, and writes the result to standard output. It's fast, scriptable, and available on virtually every Unix-like system.

The basic syntax is:

```
sed [options] 'command' file(s)
```

Or, reading from stdin:

```
cat file.txt | sed 'command'
```

---

## Core Concepts

**sed processes text line by line.** For each line, it loads the line into a buffer called the *pattern space*, applies your commands, then prints the pattern space (unless told otherwise).

**Commands are applied in order.** If you give multiple commands, each runs on the result of the previous one for every line.

**sed does not modify files by default.** Output goes to stdout. To edit files in-place, use the `-i` flag.

---

## The Substitute Command: `s`

The `s` command is the most commonly used sed feature. It replaces text matching a pattern with a replacement string.

```
s/pattern/replacement/flags
```

**Basic replacement** — change the first occurrence of "foo" on each line:

```bash
sed 's/foo/bar/' file.txt
```

**Global replacement** — change all occurrences per line using the `g` flag:

```bash
sed 's/foo/bar/g' file.txt
```

**Case-insensitive matching** using the `I` flag (GNU sed):

```bash
sed 's/foo/bar/gI' file.txt
```

**Replace only the Nth occurrence** using a number flag:

```bash
sed 's/foo/bar/2'   # replace second occurrence only
```

**The delimiter doesn't have to be `/`.** When replacing paths or URLs, a different delimiter avoids messy escaping:

```bash
sed 's|/old/path|/new/path|g' file.txt
sed 's#https://old.com#https://new.com#g' file.txt
```

---

## Backreferences

Parentheses capture groups in the pattern, and you reference them in the replacement with `\1`, `\2`, etc. In basic regex mode, parentheses must be escaped; in extended regex mode (`-E`), they are not.

```bash
# Swap first and last name (basic regex)
sed 's/\(Jane\) \(Doe\)/\2 \1/' file.txt

# Same thing with extended regex (-E)
sed -E 's/(Jane) (Doe)/\2 \1/' file.txt

# Wrap every word in brackets
sed -E 's/([a-zA-Z]+)/[\1]/g' file.txt
```

The `&` symbol in the replacement refers to the entire matched string:

```bash
# Wrap every number in parentheses
sed 's/[0-9]*/(&)/g' file.txt
```

---

## Addressing: Targeting Specific Lines

Without an address, a command applies to every line. With an address, it applies only to matching lines.

**By line number:**

```bash
sed '3s/foo/bar/' file.txt         # only line 3
sed '2,5s/foo/bar/' file.txt       # lines 2 through 5
sed '5,$s/foo/bar/' file.txt       # line 5 to end of file
```

**By pattern:**

```bash
sed '/error/s/old/new/' file.txt   # only lines containing "error"
```

**By first~step (GNU sed):**

```bash
sed '0~2s/foo/bar/' file.txt       # every even line
sed '1~2s/foo/bar/' file.txt       # every odd line
```

**Negating an address with `!`:**

```bash
sed '/error/!s/foo/bar/' file.txt  # all lines NOT containing "error"
```

**Range between two patterns:**

```bash
sed '/START/,/END/s/foo/bar/' file.txt
```

---

## Deleting Lines: `d`

```bash
sed '/pattern/d' file.txt          # delete lines matching pattern
sed '3d' file.txt                  # delete line 3
sed '2,4d' file.txt                # delete lines 2-4
sed '/^$/d' file.txt               # delete blank lines
sed '/^#/d' file.txt               # delete comment lines
```

---

## Printing Lines: `p`

By default, sed prints every line. The `p` command prints a line explicitly, often combined with `-n` (which suppresses default output) to print only what you want.

```bash
sed -n '5p' file.txt               # print only line 5
sed -n '2,8p' file.txt             # print lines 2-8
sed -n '/error/p' file.txt         # print lines matching "error"
sed -n '/START/,/END/p' file.txt   # print between two patterns
```

This is essentially a simpler `grep` for many use cases.

---

## Inserting and Appending Text: `i` and `a`

`i` inserts text *before* the addressed line; `a` appends text *after* it.

```bash
sed '3i\This line is inserted before line 3' file.txt
sed '3a\This line is appended after line 3' file.txt
sed '/pattern/a\Appended after every matching line' file.txt
```

In GNU sed, a shorter form works without the backslash:

```bash
sed '3i This line is inserted' file.txt
```

---

## Changing Lines: `c`

Replaces the entire addressed line (or range) with new text:

```bash
sed '4c\This replaces line 4 entirely' file.txt
sed '/pattern/c\Replaced line' file.txt
```

---

## Reading and Writing Files: `r` and `w`

```bash
sed '/pattern/r extra.txt' file.txt   # insert contents of extra.txt after matching line
sed -n '/pattern/w output.txt' file.txt  # write matching lines to output.txt
```

---

## Editing Files In-Place: `-i`

The `-i` flag edits a file directly instead of printing to stdout.

```bash
sed -i 's/old/new/g' file.txt
```

To keep a backup, provide an extension:

```bash
sed -i.bak 's/old/new/g' file.txt   # original saved as file.txt.bak
```

**Note:** On macOS (BSD sed), `-i` requires an argument even for no backup — use `''`:

```bash
sed -i '' 's/old/new/g' file.txt    # macOS
```

---

## Multiple Commands

Use `-e` to chain multiple commands, or separate them with semicolons:

```bash
sed -e 's/foo/bar/g' -e 's/baz/qux/g' file.txt
sed 's/foo/bar/g; s/baz/qux/g' file.txt
```

For longer scripts, put commands in a file and use `-f`:

```bash
sed -f my-script.sed file.txt
```

---

## Transliterate: `y`

The `y` command does character-by-character substitution (similar to `tr`):

```bash
sed 'y/abc/ABC/' file.txt           # replace a→A, b→B, c→C
sed 'y/abcdefghijklmnopqrstuvwxyz/ABCDEFGHIJKLMNOPQRSTUVWXYZ/' file.txt  # uppercase
```

---

## Quitting Early: `q` and `Q`

```bash
sed '5q' file.txt                   # print up to and including line 5, then quit
sed '5Q' file.txt                   # quit before printing line 5 (GNU sed)
```

This is useful for getting the head of a large file without reading the whole thing.

---

## The Hold Space

Beyond the *pattern space*, sed has a secondary buffer called the *hold space*. You can use it to carry text across lines. The relevant commands are:

| Command | Meaning |
|---------|---------|
| `h` | Copy pattern space to hold space |
| `H` | Append pattern space to hold space |
| `g` | Copy hold space to pattern space |
| `G` | Append hold space to pattern space |
| `x` | Exchange pattern space and hold space |

**Example — reverse the order of lines in a file:**

```bash
sed -n '1!G; h; $p' file.txt
```

This works by accumulating lines into the hold space and printing only at the end.

---

## Branching and Labels

sed supports labels and branching for loop-like behavior.

```bash
:label      # define a label
b label     # branch (jump) to label unconditionally
t label     # branch to label if a substitution has been made since last line read
T label     # branch if no substitution has been made (GNU sed)
```

**Example — collapse multiple blank lines into one:**

```bash
sed '/^$/{N; /^\n$/d}' file.txt
```

**Example — join continuation lines (lines ending with `\`):**

```bash
sed ':a; /\\$/{N; s/\\\n//; ba}' file.txt
```

---

## Useful One-Liners

Here's a quick reference of practical sed idioms:

```bash
# Remove leading whitespace
sed 's/^[[:space:]]*//' file.txt

# Remove trailing whitespace
sed 's/[[:space:]]*$//' file.txt

# Remove both leading and trailing whitespace
sed 's/^[[:space:]]*//; s/[[:space:]]*$//' file.txt

# Double-space a file
sed 'G' file.txt

# Number each line
sed = file.txt | sed 'N; s/\n/\t/'

# Print lines between (and including) two patterns
sed -n '/START/,/END/p' file.txt

# Delete trailing blank lines at end of file
sed -e :a -e '/^\n*$/{$d;N;ba}' file.txt

# Strip HTML tags
sed 's/<[^>]*>//g' file.txt

# Print every other line
sed -n '1~2p' file.txt

# Extract lines 10-20
sed -n '10,20p' file.txt

# Comment out lines matching a pattern
sed '/pattern/s/^/#/' file.txt
```

---

## Regular Expression Quick Reference

sed uses basic regular expressions (BRE) by default. Use `-E` for extended regular expressions (ERE), which have cleaner syntax.

| Pattern | Meaning |
|---------|---------|
| `.` | Any character except newline |
| `*` | Zero or more of the preceding |
| `^` | Start of line |
| `$` | End of line |
| `[abc]` | Any character in the set |
| `[^abc]` | Any character not in the set |
| `\+` | One or more (BRE) / `+` (ERE) |
| `\?` | Zero or one (BRE) / `?` (ERE) |
| `\{n,m\}` | n to m repetitions (BRE) / `{n,m}` (ERE) |
| `\(…\)` | Capture group (BRE) / `(…)` (ERE) |
| `[[:alpha:]]` | POSIX class: letters |
| `[[:digit:]]` | POSIX class: digits |
| `[[:space:]]` | POSIX class: whitespace |
| `[[:alnum:]]` | POSIX class: alphanumeric |

---

## Common Options

### POSIX Standard Options

These work on every POSIX-compliant `sed` implementation (GNU, BSD, macOS, etc.):

| Option | Description |
|--------|-------------|
| `-n` | Suppress automatic printing of the pattern space; only print when explicitly told to (e.g., with `p`) |
| `-e script` | Add an inline command; can be used multiple times to chain commands |
| `-f file` | Read sed commands from a script file instead of the command line |

### Widely Supported Options

Supported by both GNU and BSD sed, though syntax may differ slightly:

| Option | Description |
|--------|-------------|
| `-i[suffix]` | Edit file in-place; optional suffix creates a backup (e.g., `-i.bak`). On macOS, an argument is always required — use `-i ''` for no backup |
| `-E` | Use extended regular expressions (ERE), allowing `+`, `?`, `()` without backslash escaping. POSIX standard since 2008 |

### GNU sed Only Options

These options are specific to GNU sed (the version found on most Linux systems):

| Option | Description |
|--------|-------------|
| `-r` | Alias for `-E`; enables extended regular expressions. `-E` is preferred for portability |
| `-z` | Use null byte (`\0`) as the line delimiter instead of newline; useful for processing NUL-separated input (e.g., `find -print0`) |
| `-s` | Treat each input file as separate; line numbers and ranges reset at the start of each file |
| `-l N` | Specify the line-wrap length for the `l` command (which visually displays non-printing characters) |
| `--sandbox` | Disable the `e`, `r`, and `w` commands to prevent file I/O; useful for safely running untrusted sed scripts |
| `--follow-symlinks` | When using `-i`, follow symlinks and edit the target file rather than replacing the symlink itself |
| `--posix` | Disable all GNU extensions for strict POSIX compliance; useful for writing maximally portable scripts |
| `--debug` | Print annotated output showing exactly how sed processes each command and line; invaluable for debugging complex scripts |

---

## Tips and Gotchas

**Escaping special characters.** Characters like `.`, `*`, `[`, `\`, `/`, `^`, and `$` have special meaning in regex. To match them literally, escape with `\`. In the replacement string, `&` and `\` also need escaping.

**Newlines in patterns.** By default, sed works one line at a time and `.` does not match newlines. To match across lines, use the `N` command to pull the next line into the pattern space.

**GNU sed vs. BSD sed.** Linux systems typically use GNU sed; macOS ships with BSD sed. Some features (like `\+`, `-E`, `-i ''`) behave differently. When writing portable scripts, stick to basic features and test on both.

**Performance on large files.** For very large files, using a narrow address (like a line number or early-terminating pattern) is much faster than letting sed scan the whole file. Similarly, `sed -n '10q'` is faster than `head -10` for huge files because it stops early.

---

## Further Reading

- `man sed` — the authoritative reference on your system
- GNU sed manual: https://www.gnu.org/software/sed/manual/sed.html
- `sed` one-liners collection: https://edoras.sdsu.edu/doc/sed-oneliners.html
