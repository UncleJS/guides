# A Beginner's Guide to `grep`

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)


`grep` is one of the most powerful and commonly used command-line tools in Unix/Linux systems. It searches for patterns within text — whether in files, command output, or streams — and prints matching lines. This guide will take you from zero to confident with `grep`.


## Table of Contents

- [What is grep?](#what-is-grep)
- [Basic Syntax](#basic-syntax)
- [Your First grep Commands](#your-first-grep-commands)
- [Common Options](#common-options)
  - [`-i`: Case-Insensitive Search](#-i-case-insensitive-search)
  - [`-v`: Invert Match (Exclude Lines)](#-v-invert-match-exclude-lines)
  - [`-n`: Show Line Numbers](#-n-show-line-numbers)
  - [`-c`: Count Matching Lines](#-c-count-matching-lines)
  - [`-l`: List Filenames Only](#-l-list-filenames-only)
  - [`-L`: List Files with No Match](#-l-list-files-with-no-match)
  - [`-w`: Match Whole Words Only](#-w-match-whole-words-only)
  - [`-x`: Match Whole Lines Only](#-x-match-whole-lines-only)
  - [`-o`: Print Only the Matched Part](#-o-print-only-the-matched-part)
  - [`-q`: Quiet Mode (No Output)](#-q-quiet-mode-no-output)
  - [`-m NUM`: Stop After NUM Matches](#-m-num-stop-after-num-matches)
- [Working with Multiple Files](#working-with-multiple-files)
- [Regular Expressions](#regular-expressions)
  - [Anchors](#anchors)
  - [The Dot (Any Character)](#the-dot-any-character)
  - [Character Classes](#character-classes)
  - [Repetition Quantifiers](#repetition-quantifiers)
  - [Escaping Special Characters](#escaping-special-characters)
- [Extended Regular Expressions](#extended-regular-expressions)
  - [Alternation (OR)](#alternation-or)
  - [Grouping with Parentheses](#grouping-with-parentheses)
  - [Cleaner Quantifiers](#cleaner-quantifiers)
- [Searching Recursively](#searching-recursively)
  - [Including and Excluding Files](#including-and-excluding-files)
- [Context Lines](#context-lines)
  - [`-A NUM`: Lines After Match](#-a-num-lines-after-match)
  - [`-B NUM`: Lines Before Match](#-b-num-lines-before-match)
  - [`-C NUM`: Lines Before and After](#-c-num-lines-before-and-after)
- [Piping and Combining with Other Commands](#piping-and-combining-with-other-commands)
- [Practical Examples](#practical-examples)
  - [Find all TODO comments in a codebase](#find-all-todo-comments-in-a-codebase)
  - [Search for a specific IP address in logs](#search-for-a-specific-ip-address-in-logs)
  - [Find lines with email addresses](#find-lines-with-email-addresses)
  - [Check if a service is running](#check-if-a-service-is-running)
  - [Find files modified by a specific user in git](#find-files-modified-by-a-specific-user-in-git)
  - [Find lines containing a digit from 1 to 9](#find-lines-containing-a-digit-from-1-to-9)
  - [Find blank lines](#find-blank-lines)
  - [Extract unique IP addresses from a log](#extract-unique-ip-addresses-from-a-log)
- [Understanding Exit Codes](#understanding-exit-codes)
- [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)
- [Tips and Gotchas](#tips-and-gotchas)

---
## What is grep?

The name **grep** stands for **G**lobal **R**egular **E**xpression **P**rint. It was originally written for Unix in 1973 and remains indispensable today.

At its core, `grep` does one thing: it looks at text line by line and prints any line that matches a pattern you provide.

There are a few variants of grep you may encounter:

- `grep` — the standard version
- `grep -E` — supports extended regular expressions (historically also available as `egrep`)
- `grep -F` — treats the pattern as a fixed string, not a regex (historically `fgrep`)

> **Note:** `egrep` and `fgrep` are deprecated. Since GNU grep 3.8 they print a warning (`egrep: warning: egrep is obsolescent; using grep -E`). Use `grep -E` and `grep -F` instead.

---


[↑ Back to TOC](#table-of-contents)

## Basic Syntax

```
grep [OPTIONS] PATTERN [FILE...]
```

- **PATTERN** — the text or regular expression you want to search for
- **FILE** — one or more files to search in (optional if piping input)
- **OPTIONS** — flags that modify grep's behavior

---


[↑ Back to TOC](#table-of-contents)

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
```

grep matches the pattern *anywhere* in the line by default, so a line such as `pineapple` would match too.

**Search the system log:**
```bash
journalctl | grep -i "error"          # Fedora/RHEL (systemd journal)
sudo grep -i "error" /var/log/messages   # Fedora/RHEL with rsyslog installed
sudo grep -i "error" /var/log/syslog     # Debian/Ubuntu
```

**Search in output from another command:**
```bash
ps aux | grep "firefox"
```

---


[↑ Back to TOC](#table-of-contents)

## Common Options

These are the flags you'll reach for most often.


[↑ Back to TOC](#table-of-contents)

### `-i`: Case-Insensitive Search

```bash
grep -i "apple" fruits.txt
```
Matches `apple`, `Apple`, `APPLE`, etc.


[↑ Back to TOC](#table-of-contents)

### `-v`: Invert Match (Exclude Lines)

```bash
grep -v "apple" fruits.txt
```
Prints every line that does *not* contain "apple".


[↑ Back to TOC](#table-of-contents)

### `-n`: Show Line Numbers

```bash
grep -n "apple" fruits.txt
```
Output:
```
1:apple
```


[↑ Back to TOC](#table-of-contents)

### `-c`: Count Matching Lines

```bash
grep -c "apple" fruits.txt
```
Output:
```
1
```

Note that `-c` counts matching *lines*, not individual matches: a line containing "apple" twice still counts once.


[↑ Back to TOC](#table-of-contents)

### `-l`: List Filenames Only

When searching multiple files, show only the names of files that contain a match:
```bash
grep -l "error" *.log
```


[↑ Back to TOC](#table-of-contents)

### `-L`: List Files with No Match

```bash
grep -L "error" *.log
```


[↑ Back to TOC](#table-of-contents)

### `-w`: Match Whole Words Only

```bash
grep -w "apple" fruits.txt
```
This matches the word `apple` (also in a line such as `red apple pie`) but NOT `pineapple` or `apples`, where "apple" is only part of a longer word.


[↑ Back to TOC](#table-of-contents)

### `-x`: Match Whole Lines Only

```bash
grep -x "apple" fruits.txt
```
Only matches a line that is *exactly* "apple" with nothing else on it.


[↑ Back to TOC](#table-of-contents)

### `-o`: Print Only the Matched Part

```bash
grep -o "ap[a-z]*" fruits.txt
```
Output:
```
apple
apricot
ape
```
Instead of printing the whole line, only the matching text is printed (note `ape` from inside `grape`).


[↑ Back to TOC](#table-of-contents)

### `-q`: Quiet Mode (No Output)

Useful in scripts — grep returns exit code `0` (success) if a match is found, `1` if not:
```bash
grep -q "apple" fruits.txt && echo "Found it!"
```


[↑ Back to TOC](#table-of-contents)

### `-m NUM`: Stop After NUM Matches

```bash
grep -m 2 "apple" fruits.txt
```
Stops searching after finding 2 matches.

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

## Regular Expressions

grep's real power comes from **regular expressions** (regex) — a mini-language for describing patterns.


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

### The Dot (Any Character)

Matches any single character (except a newline):

```bash
grep "ap.le" fruits.txt     # Matches "apple", "ap_le", "ap1le", etc.
```


[↑ Back to TOC](#table-of-contents)

### Character Classes

Match any one character from a set:

```bash
grep "[aeiou]" fruits.txt   # Lines containing any vowel
grep "[a-z]" fruits.txt     # Lines containing any lowercase letter
grep "[^aeiou]" fruits.txt  # Lines containing a non-vowel character (^ negates inside [])
```


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

### Escaping Special Characters

To search for a literal dot, parenthesis, or other special regex character, escape it with `\`:

```bash
grep "\." file.txt          # Literal dot
grep '\$' file.txt          # Literal dollar sign
grep -F '$' file.txt        # Same, as a fixed string
```

Use single quotes here. Inside double quotes the shell turns `"\$"` into a bare `$`, which grep reads as the end-of-line anchor, so it matches every line.

---


[↑ Back to TOC](#table-of-contents)

## Extended Regular Expressions

Use `grep -E` (not the deprecated `egrep`) to unlock extended regex, which has cleaner syntax for advanced patterns.


[↑ Back to TOC](#table-of-contents)

### Alternation (OR)

Match one pattern OR another:

```bash
grep -E "apple|banana" fruits.txt
```


[↑ Back to TOC](#table-of-contents)

### Grouping with Parentheses

Group parts of a pattern:

```bash
grep -E "(apple|grape)s?" fruits.txt    # "apple", "apples", "grape", "grapes"
```


[↑ Back to TOC](#table-of-contents)

### Cleaner Quantifiers

With `-E`, you don't need backslashes before `+`, `?`, `{`, `}`:

```bash
grep -E "ap+le" fruits.txt       # One or more "p"
grep -E "colou?r" file.txt       # "color" or "colour"
grep -E "[0-9]{3}-[0-9]{4}" file.txt   # Phone number pattern like 555-1234
```

---


[↑ Back to TOC](#table-of-contents)

## Searching Recursively

To search through all files in a directory and its subdirectories, use `-r` (recursive):

```bash
grep -r "TODO" ./my-project/
```

Combine with `-l` to just get filenames:
```bash
grep -rl "TODO" ./my-project/
```


[↑ Back to TOC](#table-of-contents)

### Including and Excluding Files

Narrow recursive searches to specific file types:

```bash
grep -r --include="*.py" "import os" ./
grep -r --exclude="*.log" "error" ./
grep -r --exclude-dir=".git" "password" ./
```

---


[↑ Back to TOC](#table-of-contents)

## Context Lines

Sometimes you want to see the lines *around* a match for context.


[↑ Back to TOC](#table-of-contents)

### `-A NUM`: Lines After Match

```bash
grep -A 3 "error" logfile.txt    # Show 3 lines after each match
```


[↑ Back to TOC](#table-of-contents)

### `-B NUM`: Lines Before Match

```bash
grep -B 3 "error" logfile.txt    # Show 3 lines before each match
```


[↑ Back to TOC](#table-of-contents)

### `-C NUM`: Lines Before and After

```bash
grep -C 3 "error" logfile.txt    # Show 3 lines before AND after each match
```

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

## Practical Examples


[↑ Back to TOC](#table-of-contents)

### Find all TODO comments in a codebase
```bash
grep -rn "TODO" ./src/ --include="*.js"
```


[↑ Back to TOC](#table-of-contents)

### Search for a specific IP address in logs
```bash
grep "192\.168\.1\.1" access.log
```


[↑ Back to TOC](#table-of-contents)

### Find lines with email addresses
```bash
grep -E "[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}" contacts.txt
```


[↑ Back to TOC](#table-of-contents)

### Check if a service is running
```bash
ps aux | grep -v grep | grep "httpd"
```
(The `-v grep` part excludes the grep process itself from results. The Apache service is `httpd` on Fedora/RHEL and `apache2` on Debian/Ubuntu.)

A simpler alternative that never matches itself:
```bash
pgrep -a httpd
```


[↑ Back to TOC](#table-of-contents)

### Find files modified by a specific user in git
```bash
git log --author="yourname" --name-only --pretty=format: | sort -u
```
`--author` does the filtering (it takes a regex, like grep), and `--pretty=format:` hides the commit headers so only file names remain.


[↑ Back to TOC](#table-of-contents)

### Find lines containing a digit from 1 to 9
```bash
grep "[1-9]" data.txt      # Any line containing a digit 1-9 (also matches "10", "250", ...)
grep -w "[1-9]" data.txt   # Only a standalone single-digit number from 1 to 9
```


[↑ Back to TOC](#table-of-contents)

### Find blank lines
```bash
grep -c "^$" file.txt    # Count blank lines
grep -v "^$" file.txt    # Remove blank lines from output
```


[↑ Back to TOC](#table-of-contents)

### Extract unique IP addresses from a log
```bash
grep -oE "[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}" access.log | sort -u
```

---


[↑ Back to TOC](#table-of-contents)

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


[↑ Back to TOC](#table-of-contents)

## Quick Reference Cheat Sheet

```
BASIC
  grep "pattern" file          Search for pattern in file
  grep -i "pattern" file       Case-insensitive
  grep -v "pattern" file       Invert match (exclude)
  grep -n "pattern" file       Show line numbers
  grep -c "pattern" file       Count matching lines
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


[↑ Back to TOC](#table-of-contents)

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

**Fixed strings.** Use `grep -F` (fixed string) when you don't need regex. It saves you from escaping characters like `.`, `*` or `[`, and it can be faster with many patterns (for example `grep -F -f words.txt`). For a single plain word GNU grep is already fast, so the speed gain is usually small.

**Binary files.** By default grep prints a message instead of the matching lines for binary files (`grep: X: binary file matches` in GNU grep 3.5+; older versions printed `Binary file X matches`). Use `-a` to treat binary as text, or `--include` to restrict to text file types.

**GNU vs BSD grep.** On macOS, `grep` is BSD grep, which behaves slightly differently from GNU grep on Linux. Most options covered here work on both, but some advanced flags may differ. Install GNU grep on macOS via Homebrew (`brew install grep`) if needed.

---

With these fundamentals, you're well-equipped to use `grep` effectively in your day-to-day work. The best way to get comfortable is to practice — next time you're looking for something in a file or log, reach for `grep` instead of opening a text editor.

[↑ Back to TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
