# A Beginner's Guide to `grep`

`grep` is one of the most powerful and commonly used command-line tools in Unix/Linux systems. It searches for patterns within text — whether in files, command output, or streams — and prints matching lines. This guide will take you from zero to confident with `grep`.

---

## Table of Contents

1. [What is grep?](#what-is-grep)
2. [Basic Syntax](#basic-syntax)
3. [Your First grep Commands](#your-first-grep-commands)
4. [Common Options](#common-options)
5. [Working with Multiple Files](#working-with-multiple-files)
6. [Regular Expressions](#regular-expressions)
7. [Extended Regular Expressions](#extended-regular-expressions)
8. [Searching Recursively](#searching-recursively)
9. [Piping and Combining with Other Commands](#piping-and-combining-with-other-commands)
10. [Practical Examples](#practical-examples)
11. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## What is grep?

The name **grep** stands for **G**lobal **R**egular **E**xpression **P**rint. It was originally written for Unix in 1973 and remains indispensable today.

At its core, `grep` does one thing: it looks at text line by line and prints any line that matches a pattern you provide.

There are a few variants of grep you may encounter:

- `grep` — the standard version
- `egrep` — supports extended regular expressions (same as `grep -E`)
- `fgrep` — treats the pattern as a fixed string, not a regex (same as `grep -F`)

---

## Basic Syntax

```
grep [OPTIONS] PATTERN [FILE...]
```

- **PATTERN** — the text or regular expression you want to search for
- **FILE** — one or more files to search in (optional if piping input)
- **OPTIONS** — flags that modify grep's behavior

---

## Your First grep Commands

Let's start simple. Suppose you have a file called `fruits.txt`:

```
apple
banana
cherry
apricot
blueberry
grape
```

**Find lines containing "apple":**
```bash
grep "apple" fruits.txt
```
Output:
```
apple
apricot
```

Both lines match because `apricot` contains the letters "apple" — grep matches *anywhere* in the line by default.

**Search in a log file:**
```bash
grep "error" /var/log/syslog
```

**Search in output from another command:**
```bash
ps aux | grep "firefox"
```

---

## Common Options

These are the flags you'll reach for most often.

### `-i` — Case-Insensitive Search

```bash
grep -i "apple" fruits.txt
```
Matches `apple`, `Apple`, `APPLE`, etc.

### `-v` — Invert Match (exclude lines)

```bash
grep -v "apple" fruits.txt
```
Prints every line that does *not* contain "apple".

### `-n` — Show Line Numbers

```bash
grep -n "apple" fruits.txt
```
Output:
```
1:apple
4:apricot
```

### `-c` — Count Matching Lines

```bash
grep -c "apple" fruits.txt
```
Output:
```
2
```

### `-l` — List Filenames Only

When searching multiple files, show only the names of files that contain a match:
```bash
grep -l "error" *.log
```

### `-L` — List Files with NO Match

```bash
grep -L "error" *.log
```

### `-w` — Match Whole Words Only

```bash
grep -w "apple" fruits.txt
```
This matches `apple` but NOT `apricot` or `pineapple`.

### `-x` — Match Whole Lines Only

```bash
grep -x "apple" fruits.txt
```
Only matches a line that is *exactly* "apple" with nothing else on it.

### `-o` — Print Only the Matched Part

```bash
grep -o "ap[a-z]*" fruits.txt
```
Instead of printing the whole line, only the matching text is printed.

### `-q` — Quiet Mode (no output)

Useful in scripts — grep returns exit code `0` (success) if a match is found, `1` if not:
```bash
grep -q "apple" fruits.txt && echo "Found it!"
```

### `-m NUM` — Stop After NUM Matches

```bash
grep -m 2 "apple" fruits.txt
```
Stops searching after finding 2 matches.

---

## Working with Multiple Files

`grep` can search through several files at once:

```bash
grep "error" file1.log file2.log file3.log
```

Use wildcards to search all files of a type:
```bash
grep "TODO" *.py
```

When multiple files are searched, grep prefixes each result with the filename:
```
file1.log:2024-01-01 ERROR: connection failed
file2.log:2024-01-02 ERROR: timeout
```

---

## Regular Expressions

grep's real power comes from **regular expressions** (regex) — a mini-language for describing patterns.

### Anchors

| Pattern | Meaning |
|---------|---------|
| `^` | Start of a line |
| `$` | End of a line |

```bash
grep "^apple" fruits.txt    # Lines starting with "apple"
grep "berry$" fruits.txt    # Lines ending with "berry"
grep "^$" file.txt          # Empty lines
```

### The Dot `.`

Matches any single character (except a newline):

```bash
grep "ap.le" fruits.txt     # Matches "apple", "ap_le", "ap1le", etc.
```

### Character Classes `[...]`

Match any one character from a set:

```bash
grep "[aeiou]" fruits.txt   # Lines containing any vowel
grep "[a-z]" fruits.txt     # Lines containing any lowercase letter
grep "[^aeiou]" fruits.txt  # Lines containing a non-vowel character (^ negates inside [])
```

### Repetition Quantifiers

| Quantifier | Meaning |
|------------|---------|
| `*` | 0 or more of the preceding character |
| `\+` | 1 or more (use `+` with `-E`) |
| `\?` | 0 or 1 (use `?` with `-E`) |
| `\{n\}` | Exactly n times |
| `\{n,m\}` | Between n and m times |

```bash
grep "ap*le" fruits.txt     # "ale", "aple", "apple", "appple", etc.
grep "ap\+le" fruits.txt    # "aple", "apple", etc. (at least one "p")
```

### Escaping Special Characters

To search for a literal dot, parenthesis, or other special regex character, escape it with `\`:

```bash
grep "\." file.txt          # Literal dot
grep "\$" file.txt          # Literal dollar sign
```

---

## Extended Regular Expressions

Use `grep -E` (or `egrep`) to unlock extended regex, which has cleaner syntax for advanced patterns.

### Alternation `|`

Match one pattern OR another:

```bash
grep -E "apple|banana" fruits.txt
```

### Grouping `(...)`

Group parts of a pattern:

```bash
grep -E "(apple|grape)s?" fruits.txt    # "apple", "apples", "grape", "grapes"
```

### Cleaner Quantifiers

With `-E`, you don't need backslashes before `+`, `?`, `{`, `}`:

```bash
grep -E "ap+le" fruits.txt       # One or more "p"
grep -E "colou?r" file.txt       # "color" or "colour"
grep -E "[0-9]{3}-[0-9]{4}" file.txt   # Phone number pattern like 555-1234
```

---

## Searching Recursively

To search through all files in a directory and its subdirectories, use `-r` (recursive):

```bash
grep -r "TODO" ./my-project/
```

Combine with `-l` to just get filenames:
```bash
grep -rl "TODO" ./my-project/
```

### `--include` and `--exclude`

Narrow recursive searches to specific file types:

```bash
grep -r --include="*.py" "import os" ./
grep -r --exclude="*.log" "error" ./
grep -r --exclude-dir=".git" "password" ./
```

---

## Context Lines

Sometimes you want to see the lines *around* a match for context.

### `-A NUM` — Lines After Match

```bash
grep -A 3 "error" logfile.txt    # Show 3 lines after each match
```

### `-B NUM` — Lines Before Match

```bash
grep -B 3 "error" logfile.txt    # Show 3 lines before each match
```

### `-C NUM` — Lines Before and After

```bash
grep -C 3 "error" logfile.txt    # Show 3 lines before AND after each match
```

---

## Piping and Combining with Other Commands

`grep` shines when combined with other Unix tools using pipes (`|`).

**Filter running processes:**
```bash
ps aux | grep "nginx"
```

**Find files then search in them:**
```bash
find . -name "*.txt" | xargs grep "hello"
```

**Count occurrences of an error in a log:**
```bash
cat app.log | grep "ERROR" | wc -l
```

**Search command history:**
```bash
history | grep "git commit"
```

**Chain multiple greps to narrow results:**
```bash
grep "ERROR" app.log | grep -v "timeout" | grep "database"
```

**Use with `sort` and `uniq`:**
```bash
grep "ERROR" app.log | sort | uniq -c | sort -rn
```
This counts and ranks unique error messages by frequency.

---

## Practical Examples

### Find all TODO comments in a codebase
```bash
grep -rn "TODO" ./src/ --include="*.js"
```

### Search for a specific IP address in logs
```bash
grep "192\.168\.1\.1" access.log
```

### Find lines with email addresses
```bash
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" contacts.txt
```

### Check if a service is running
```bash
ps aux | grep -v grep | grep "apache2"
```
(The `-v grep` part excludes the grep process itself from results.)

### Find files modified by a specific user in git
```bash
git log --name-only | grep "yourname"
```

### Search for lines with a number between 1 and 9
```bash
grep "[1-9]" data.txt
```

### Find blank lines
```bash
grep -c "^$" file.txt    # Count blank lines
grep -v "^$" file.txt    # Remove blank lines from output
```

### Extract unique IP addresses from a log
```bash
grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log | sort -u
```

---

## Understanding Exit Codes

`grep` returns an exit code you can use in shell scripts:

| Exit Code | Meaning |
|-----------|---------|
| `0` | One or more matches found |
| `1` | No matches found |
| `2` | An error occurred |

```bash
if grep -q "failed" results.txt; then
    echo "There were failures!"
else
    echo "All good."
fi
```

---

## Quick Reference Cheat Sheet

```
BASIC
  grep "pattern" file          Search for pattern in file
  grep -i "pattern" file       Case-insensitive
  grep -v "pattern" file       Invert match (exclude)
  grep -n "pattern" file       Show line numbers
  grep -c "pattern" file       Count matches
  grep -l "pattern" *.txt      List matching filenames
  grep -w "pattern" file       Match whole word only
  grep -x "pattern" file       Match whole line only
  grep -o "pattern" file       Print only matched part
  grep -q "pattern" file       Quiet mode (for scripts)
  grep -m 5 "pattern" file     Stop after 5 matches

REGEX
  .            Any single character
  ^            Start of line
  $            End of line
  [abc]        Any of a, b, or c
  [^abc]       NOT a, b, or c
  [a-z]        Any lowercase letter
  *            0 or more of previous
  \+           1 or more of previous
  \?           0 or 1 of previous

EXTENDED REGEX (grep -E)
  +            1 or more
  ?            0 or 1
  |            OR (alternation)
  (abc)        Grouping
  {n,m}        Between n and m times

FILES & DIRECTORIES
  grep -r "pattern" dir/       Recursive search
  grep -rl "pattern" dir/      Recursive, filenames only
  grep --include="*.py" ...    Filter by file type
  grep --exclude-dir=".git"    Exclude directory

CONTEXT
  grep -A 3 "pattern" file     3 lines after match
  grep -B 3 "pattern" file     3 lines before match
  grep -C 3 "pattern" file     3 lines before and after

PIPING
  command | grep "pattern"     Filter command output
  grep "a" f | grep -v "b"     Chain greps
```

---

## Tips and Gotchas

**Quote your patterns.** Always wrap your pattern in quotes to prevent the shell from interpreting special characters before grep sees them:
```bash
grep "hello world" file.txt   # Correct
grep hello world file.txt     # Wrong — "world" is treated as a filename
```

**Grepping for a string that starts with a dash.** Use `--` to signal the end of options:
```bash
grep -- "-v" file.txt
```

**Performance on large files.** Use `grep -F` (fixed string) when you don't need regex — it's significantly faster because it skips regex parsing.

**Binary files.** By default grep will say "Binary file X matches" for binary files. Use `-a` to treat binary as text, or `--include` to restrict to text file types.

**GNU vs BSD grep.** On macOS, `grep` is BSD grep, which behaves slightly differently from GNU grep on Linux. Most options covered here work on both, but some advanced flags may differ. Install GNU grep on macOS via Homebrew (`brew install grep`) if needed.

---

With these fundamentals, you're well-equipped to use `grep` effectively in your day-to-day work. The best way to get comfortable is to practice — next time you're looking for something in a file or log, reach for `grep` instead of opening a text editor.
