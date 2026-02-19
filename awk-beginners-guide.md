# A Beginner's Guide to AWK

AWK is a powerful text-processing language that has been a staple of Unix systems since the 1970s. Named after its creators — **A**ho, **W**einberger, and **K**ernighan — it excels at processing structured text, extracting fields, and generating reports. If you work with log files, CSVs, or any columnar data, AWK will change your life.

---

## Table of Contents

1. [What is AWK?](#what-is-awk)
2. [Basic Syntax](#basic-syntax)
3. [Running AWK](#running-awk)
4. [Records and Fields](#records-and-fields)
5. [Built-in Variables](#built-in-variables)
6. [Patterns](#patterns)
7. [Actions](#actions)
8. [Arithmetic and String Operations](#arithmetic-and-string-operations)
9. [Built-in Functions](#built-in-functions)
10. [Control Flow](#control-flow)
11. [Arrays](#arrays)
12. [BEGIN and END Blocks](#begin-and-end-blocks)
13. [Multiple Input Files](#multiple-input-files)
14. [AWK Scripts](#awk-scripts)
15. [Common Real-World Examples](#common-real-world-examples)
16. [Tips and Gotchas](#tips-and-gotchas)

---

## What is AWK?

AWK reads input line by line, splits each line into fields, and lets you apply rules to process those fields. It's not a general-purpose programming language — it's purpose-built for text transformation, filtering, and reporting.

Think of AWK as a data pipeline where you define: *"For lines that match this pattern, do this action."*

AWK is installed by default on virtually every Unix, Linux, and macOS system. You'll encounter several variants:

- **awk** — the original (often a symlink to one of the below)
- **gawk** — GNU AWK, the most feature-rich version (default on Linux)
- **mawk** — a fast, lightweight variant
- **nawk** — "new AWK", common on BSD/macOS

The examples in this guide work with all common variants unless noted.

---

## Basic Syntax

Every AWK program is made up of one or more **rules**. Each rule has the form:

```
pattern { action }
```

- **pattern** — a condition that determines whether the action runs (optional)
- **action** — what to do when the pattern matches (optional)

If you omit the pattern, the action runs on every line. If you omit the action, the default is to print the matched line.

```awk
# Print every line
{ print }

# Print lines containing "error"
/error/ { print }

# Print the first field of every line
{ print $1 }

# Print lines where field 3 is greater than 100
$3 > 100 { print }
```

---

## Running AWK

You can run AWK directly from the command line or from a script file.

### Inline (one-liner)

```bash
awk 'program' file
```

```bash
# Print the second field from each line of data.txt
awk '{ print $2 }' data.txt
```

### Piped input

```bash
echo "hello world" | awk '{ print $1 }'
# Output: hello
```

### From a script file

```bash
awk -f myscript.awk data.txt
```

### Passing variables with -v

```bash
awk -v threshold=50 '$3 > threshold { print }' data.txt
```

---

## Records and Fields

This is the heart of AWK.

### Records

By default, AWK reads one **record** (line) at a time. The record separator is stored in the variable `RS` (record separator), which defaults to a newline `\n`.

### Fields

Each record is split into **fields** by the **field separator** `FS`, which defaults to any whitespace (spaces or tabs). Fields are accessed with `$1`, `$2`, `$3`, etc.

- `$0` — the entire record (the whole line)
- `$1` — the first field
- `$2` — the second field
- `$NF` — the last field (NF = number of fields)
- `$(NF-1)` — the second-to-last field

```bash
echo "Alice 30 Engineer" | awk '{ print $1, $3 }'
# Output: Alice Engineer
```

### Changing the field separator

Use `-F` on the command line or set `FS` in a BEGIN block:

```bash
# Parse a CSV file
awk -F',' '{ print $1, $3 }' employees.csv

# Parse /etc/passwd (colon-separated)
awk -F':' '{ print $1, $7 }' /etc/passwd
```

You can use a regex as the separator:

```bash
awk -F'[,;]' '{ print $2 }' data.txt
```

### Modifying fields

You can assign to fields, which rebuilds `$0`:

```bash
echo "hello world" | awk '{ $2 = "AWK"; print $0 }'
# Output: hello AWK
```

---

## Built-in Variables

AWK has a set of useful built-in variables you can read or set.

| Variable | Description | Default |
|----------|-------------|---------|
| `FS` | Field separator | whitespace |
| `RS` | Record separator | `\n` |
| `OFS` | Output field separator | space |
| `ORS` | Output record separator | `\n` |
| `NF` | Number of fields in current record | — |
| `NR` | Total number of records read so far | — |
| `FNR` | Record number within the current file | — |
| `FILENAME` | Name of the current input file | — |

### OFS and ORS examples

```bash
# Print fields 1 and 3, separated by a comma
awk 'BEGIN { OFS="," } { print $1, $3 }' data.txt

# Print records separated by blank lines
awk 'BEGIN { ORS="\n\n" } { print }' data.txt
```

### NR and FNR

`NR` counts records across all files. `FNR` resets for each file.

```bash
# Print line numbers
awk '{ print NR, $0 }' data.txt

# Print the first line of each input file
awk 'FNR == 1 { print FILENAME, $0 }' file1.txt file2.txt
```

---

## Patterns

Patterns are conditions that control when an action runs.

### No pattern

The action runs on every line.

```bash
awk '{ print $1 }' file.txt
```

### Regular expression pattern

Enclose the regex in `/slashes/`:

```bash
# Print lines containing "ERROR"
awk '/ERROR/ { print }' app.log

# Print lines NOT containing "DEBUG"
awk '!/DEBUG/ { print }' app.log
```

### Comparison pattern

```bash
awk '$3 > 100 { print }' data.txt
awk '$1 == "Alice" { print }' data.txt
awk 'NR > 5 && NR < 10 { print }' data.txt
```

### Range pattern

Match from a start pattern to an end pattern (inclusive):

```bash
awk '/START/,/END/ { print }' file.txt
```

This prints every line from the first occurrence of "START" through the first occurrence of "END".

### Compound patterns

Use `&&` (and), `||` (or), `!` (not):

```bash
awk '$2 > 50 && $3 == "active" { print }' data.txt
awk '/error/ || /warning/ { print }' app.log
```

---

## Actions

Actions are enclosed in `{ }` and can contain multiple statements separated by semicolons or newlines.

### print vs printf

`print` adds OFS between arguments and ORS at the end.

```bash
awk '{ print $1, $2, $3 }' file.txt
```

`printf` works like C's printf — you control the format exactly:

```bash
awk '{ printf "Name: %-10s Age: %d\n", $1, $2 }' file.txt
```

Common printf format specifiers:

| Specifier | Description |
|-----------|-------------|
| `%s` | string |
| `%d` | integer |
| `%f` | float |
| `%e` | scientific notation |
| `%-10s` | left-aligned string, width 10 |
| `%05d` | zero-padded integer, width 5 |

### Redirecting output

```bash
# Write to a file
awk '{ print $1 > "output.txt" }' data.txt

# Append to a file
awk '{ print $1 >> "output.txt" }' data.txt

# Pipe to a shell command
awk '{ print $1 | "sort" }' data.txt
```

---

## Arithmetic and String Operations

### Arithmetic

AWK supports standard arithmetic operators: `+`, `-`, `*`, `/`, `%`, `^` (exponentiation).

```bash
awk '{ print $1 * $2 }' data.txt
awk '{ total += $3 } END { print total }' data.txt
```

AWK performs automatic type conversion — a string that looks like a number is treated as one in numeric contexts.

### String concatenation

Strings are concatenated simply by placing them next to each other:

```bash
awk '{ full = $1 " " $2; print full }' data.txt
```

### Comparison operators

`==`, `!=`, `<`, `>`, `<=`, `>=` work for both numbers and strings. AWK compares numerically if both operands look like numbers, otherwise as strings.

```bash
awk '$1 == "Alice" { print }' data.txt   # string comparison
awk '$2 == 30 { print }' data.txt        # numeric comparison
```

### String matching operators

```bash
# $1 matches the regex
awk '$1 ~ /^A/ { print }' data.txt

# $1 does NOT match the regex
awk '$1 !~ /^A/ { print }' data.txt
```

---

## Built-in Functions

### String functions

| Function | Description |
|----------|-------------|
| `length(s)` | Length of string s (or `$0` if no arg) |
| `substr(s, start, len)` | Substring (1-indexed) |
| `index(s, t)` | Position of t in s (0 if not found) |
| `split(s, a, sep)` | Split s into array a on sep |
| `sub(regex, repl, s)` | Replace first match in s |
| `gsub(regex, repl, s)` | Replace all matches in s |
| `match(s, regex)` | Find regex in s; sets RSTART, RLENGTH |
| `toupper(s)` | Convert to uppercase |
| `tolower(s)` | Convert to lowercase |
| `sprintf(fmt, ...)` | Format a string like printf |
| `gensub(re, repl, how, s)` | Advanced replace (gawk only) |

```bash
# Print length of each line
awk '{ print length($0) }' file.txt

# Extract characters 2-5 from field 1
awk '{ print substr($1, 2, 4) }' file.txt

# Replace "foo" with "bar" everywhere
awk '{ gsub(/foo/, "bar"); print }' file.txt

# Split a comma-delimited field into an array
awk '{ n = split($3, parts, ","); for (i=1; i<=n; i++) print parts[i] }' file.txt
```

### Numeric functions

| Function | Description |
|----------|-------------|
| `int(x)` | Truncate to integer |
| `sqrt(x)` | Square root |
| `log(x)` | Natural logarithm |
| `exp(x)` | e^x |
| `sin(x)` | Sine (radians) |
| `cos(x)` | Cosine (radians) |
| `atan2(y, x)` | Arctangent |
| `rand()` | Random number [0, 1) |
| `srand(seed)` | Seed the random number generator |

```bash
awk 'BEGIN { srand(); print int(rand() * 100) }'
```

---

## Control Flow

AWK supports familiar control flow structures.

### if / else

```bash
awk '{
    if ($3 > 100) {
        print $1, "above threshold"
    } else {
        print $1, "below threshold"
    }
}' data.txt
```

### while loop

```bash
awk '{
    i = 1
    while (i <= NF) {
        print i, $i
        i++
    }
}' data.txt
```

### for loop

```bash
# C-style for loop
awk '{
    for (i = 1; i <= NF; i++) {
        print i, $i
    }
}' data.txt

# for-in loop (iterates over array keys)
awk '{
    for (key in counts) {
        print key, counts[key]
    }
}' data.txt
```

### do-while loop

```bash
awk 'BEGIN {
    i = 1
    do {
        print i
        i++
    } while (i <= 5)
}'
```

### next and exit

`next` skips to the next record (like `continue` for the main loop).
`exit` stops processing and jumps to the END block.

```bash
# Skip blank lines
awk 'NF == 0 { next } { print }' file.txt

# Stop after processing 100 records
awk 'NR > 100 { exit } { print }' file.txt
```

---

## Arrays

AWK arrays are associative (like dictionaries/hash maps). Keys can be strings or numbers.

```bash
# Count occurrences of each value in field 1
awk '{ count[$1]++ } END { for (name in count) print name, count[name] }' data.txt
```

### Checking if a key exists

```bash
awk '{
    if ("Alice" in count) print "Alice has entries"
}' data.txt
```

### Deleting elements

```bash
delete count["Alice"]   # delete one element
delete count            # delete the entire array
```

### Multi-dimensional arrays

AWK simulates multi-dimensional arrays with concatenated keys:

```bash
awk '{ grid[$1][$2]++ }' data.txt          # gawk only
awk '{ grid[$1, $2]++ }' data.txt           # all AWK (uses SUBSEP as separator)
```

---

## BEGIN and END Blocks

`BEGIN` runs once before any input is read. `END` runs once after all input has been processed. Neither requires input.

```bash
awk '
BEGIN {
    print "Starting report..."
    FS = ","
    total = 0
}
{
    total += $3
}
END {
    print "Total:", total
    print "Lines processed:", NR
}
' sales.csv
```

You can have multiple BEGIN and END blocks — they execute in order.

```bash
# Use AWK as a pure calculator
awk 'BEGIN { print 2^10 }'
# Output: 1024
```

---

## Multiple Input Files

AWK handles multiple files naturally:

```bash
awk '{ print FILENAME, NR, $0 }' file1.txt file2.txt
```

`NR` is the total record count across all files. `FNR` resets per file.

A common pattern to do something only for the first file:

```bash
awk 'FNR == NR { store[$1] = $2; next } { print $1, store[$1] }' lookup.txt data.txt
```

This classic idiom loads `lookup.txt` into an array (when `FNR == NR`, we're on the first file), then uses it while processing `data.txt`.

---

## AWK Scripts

For anything beyond a one-liner, put your AWK code in a file:

```awk
#!/usr/bin/awk -f
# report.awk — Summarize sales data

BEGIN {
    FS = ","
    OFS = "\t"
    print "Region", "Total Sales"
}

NR > 1 {    # skip header row
    region[$1] += $4
}

END {
    for (r in region) {
        printf "%-15s %10.2f\n", r, region[r]
    }
}
```

Run it:

```bash
awk -f report.awk sales.csv
# or make it executable:
chmod +x report.awk
./report.awk sales.csv
```

---

## Common Real-World Examples

### Print specific columns

```bash
awk '{ print $1, $3 }' file.txt
```

### Print line numbers

```bash
awk '{ print NR": "$0 }' file.txt
```

### Sum a column

```bash
awk '{ sum += $2 } END { print "Total:", sum }' data.txt
```

### Calculate average

```bash
awk '{ sum += $2 } END { print "Average:", sum/NR }' data.txt
```

### Filter rows by value

```bash
awk -F',' '$4 > 1000 { print }' sales.csv
```

### Print lines between two patterns

```bash
awk '/BEGIN_SECTION/,/END_SECTION/' file.txt
```

### Remove duplicate lines (maintaining order)

```bash
awk '!seen[$0]++' file.txt
```

### Print the last field of every line

```bash
awk '{ print $NF }' file.txt
```

### Reverse the order of fields

```bash
awk '{ for (i=NF; i>=1; i--) printf "%s%s", $i, (i>1 ? OFS : ORS) }' file.txt
```

### Count lines matching a pattern

```bash
awk '/error/ { count++ } END { print count }' app.log
```

### Extract unique values from a column

```bash
awk '!seen[$1]++ { print $1 }' data.txt
```

### Add a header and footer to output

```bash
awk 'BEGIN { print "=== Report ===" } { print } END { print "=== End ===" }' file.txt
```

### Compute word frequency

```bash
awk '{
    for (i=1; i<=NF; i++) freq[tolower($i)]++
} END {
    for (w in freq) print freq[w], w
}' document.txt | sort -rn | head -20
```

### Parse Apache/Nginx access logs

```bash
# Print IP address and status code from a common log format
awk '{ print $1, $9 }' access.log | sort | uniq -c | sort -rn
```

### Join two files on a common key

```bash
# Load file1 as a lookup table, then enrich file2
awk -F',' 'FNR==NR { map[$1]=$2; next } { print $0, map[$1] }' file1.csv file2.csv
```

### Find the maximum value in a column

```bash
awk 'NR==1 || $3 > max { max=$3 } END { print "Max:", max }' data.txt
```

---

## Tips and Gotchas

**AWK arrays are unordered.** When you iterate over an array with `for (key in array)`, the order is not guaranteed. Pipe through `sort` if you need sorted output.

**Field separator vs output field separator.** `FS` controls how input is split; `OFS` controls how output is assembled when you use `print $1, $2`. If you modify a field and print `$0`, AWK rebuilds it with OFS between fields.

```bash
echo "a:b:c" | awk -F':' 'BEGIN{OFS=","} {$1=$1; print $0}'
# Output: a,b,c
# The $1=$1 trick forces AWK to rebuild $0 with OFS
```

**0 vs empty string.** In AWK, uninitialized variables are both 0 (numerically) and `""` (as a string). This is usually convenient but can cause unexpected comparisons.

**Regex constants vs variables.** `/regex/` is a regex literal. If you want to match against a variable, use `~`:

```bash
awk -v pat="error" '$0 ~ pat { print }' file.txt
```

**Integer arithmetic.** AWK uses floating point internally. For large integers, results may lose precision. Use `printf "%d"` to truncate.

**Single quotes in one-liners.** AWK programs are almost always wrapped in single quotes on the command line to prevent the shell from interpreting `$`, `*`, etc. If you need a literal single quote inside the program, you have to end the quoting, escape it, and resume: `'...'\"'\"'...'` — or better yet, use a script file.

**Multiline records.** Set `RS=""` to treat blank-line-delimited paragraphs as single records. Each paragraph becomes one record, and newlines within it become part of `$0`.

```bash
awk 'BEGIN { RS=""; FS="\n" } { print "Record:", NR; print "Lines:", NF }' paragraphs.txt
```

---

## Quick Reference Card

```
awk [options] 'program' file(s)

Options:
  -F sep       Set field separator
  -v var=val   Set variable
  -f file      Read program from file

Special variables:
  $0           Entire record
  $1..$NF      Individual fields
  NF           Number of fields
  NR           Record number (total)
  FNR          Record number (per file)
  FS / OFS     Input / output field separator
  RS / ORS     Input / output record separator
  FILENAME     Current filename

Patterns:
  /regex/      Match regex
  !/regex/     Not match
  expr         Truthy expression
  p1, p2       Range pattern
  BEGIN        Before first record
  END          After last record

Useful one-liners:
  awk '{print NR, $0}'        Add line numbers
  awk 'NR%2==0'               Print even lines
  awk '!seen[$0]++'           Remove duplicates
  awk '{sum+=$1} END{print sum}'   Sum column 1
  awk 'length($0) > 80'       Lines longer than 80 chars
  awk '{print $NF}'           Print last field
```

---

*Happy AWKing! Once you internalize the pattern-action model, you'll find yourself reaching for AWK constantly for quick data wrangling tasks that would take far more code in any other language.*
