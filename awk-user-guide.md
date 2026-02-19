# AWK: A Comprehensive User Guide for Absolute Beginners

## Table of Contents

1. [What Is AWK?](#what-is-awk)
2. [Getting Started](#getting-started)
3. [How AWK Thinks: The Record/Field Model](#how-awk-thinks)
4. [Basic Syntax](#basic-syntax)
5. [Patterns](#patterns)
6. [Actions](#actions)
7. [Built-in Variables](#built-in-variables)
8. [Operators](#operators)
9. [Control Flow](#control-flow)
10. [Arrays](#arrays)
11. [Built-in Functions](#built-in-functions)
12. [User-Defined Functions](#user-defined-functions)
13. [BEGIN and END Blocks](#begin-and-end-blocks)
14. [Working with Multiple Files](#working-with-multiple-files)
15. [AWK One-Liners Cookbook](#awk-one-liners-cookbook)
16. [Real-World Examples](#real-world-examples)
17. [Tips, Tricks, and Gotchas](#tips-tricks-and-gotchas)

---

## 1. What Is AWK?

AWK is a text-processing language that has been part of Unix systems since 1977. Its name comes from the initials of its three creators: **A**lfred Aho, **P**eter **W**einberger, and **B**rian **K**ernighan. Despite its age, AWK remains one of the most powerful and efficient tools for processing structured text — things like log files, CSV data, reports, and columnar output from other commands.

AWK excels at tasks like:

- Extracting specific columns from tabular data
- Filtering lines based on conditions
- Computing sums, averages, and counts
- Reformatting data from one structure to another
- Generating reports from raw data

Think of AWK as a domain-specific language purpose-built for the pattern: *read a line, check if it matches a condition, do something with it*.

---

## 2. Getting Started

AWK is pre-installed on virtually every Linux and macOS system. On Windows, you can get it via WSL, Git Bash, or Cygwin. There are also several implementations — `gawk` (GNU AWK) is the most feature-rich and is the default on most Linux systems. This guide focuses on standard AWK features that work everywhere, noting `gawk`-specific features when used.

To check if AWK is available and see its version:

```bash
awk --version
gawk --version
```

**Your first AWK command:**

```bash
echo "Hello World" | awk '{ print $2 }'
```

Output: `World`

Congratulations — you just ran AWK. It read one line of input, split it into words, and printed the second word. Let's understand why.

---

## 3. How AWK Thinks: The Record/Field Model

AWK's entire worldview is built around two concepts: **records** and **fields**.

**Records** are lines of input. By default, AWK reads your input one line at a time. Each line is one record. (You can change this, but for now: one line = one record.)

**Fields** are the pieces within each record. By default, AWK splits each record on whitespace (spaces and tabs). These pieces are numbered starting at 1.

Consider this line of text:

```
Alice   30   Engineer   New York
```

AWK sees:

| Variable | Value      |
|----------|------------|
| `$0`     | The whole line: `Alice   30   Engineer   New York` |
| `$1`     | `Alice`    |
| `$2`     | `30`       |
| `$3`     | `Engineer` |
| `$4`     | `New`      |
| `$5`     | `York`     |

Notice that "New York" gets split because AWK sees it as two fields separated by a space. We'll learn how to handle this later using custom field separators.

`NF` is a special variable that always holds the **number of fields** in the current record. So `$NF` always refers to the **last field** — very handy.

---

## 4. Basic Syntax

The fundamental structure of an AWK program is:

```
awk 'pattern { action }' input_file
```

Or with multiple rules:

```
awk '
  pattern1 { action1 }
  pattern2 { action2 }
  pattern3 { action3 }
' input_file
```

A few important points about this structure:

- The AWK program is enclosed in single quotes to protect it from the shell.
- The `pattern` is optional. If omitted, the action runs on every record.
- The `{ action }` is optional. If omitted, AWK prints the whole matching line (same as `{ print $0 }`).
- If both are omitted... you have nothing.

**Running AWK:**

```bash
# From a file
awk '{ print $1 }' data.txt

# From a pipeline
cat data.txt | awk '{ print $1 }'

# Inline data (using echo)
echo "one two three" | awk '{ print $2 }'

# AWK program saved in a file
awk -f my_program.awk data.txt

# Passing variables from the shell
awk -v threshold=100 '$2 > threshold { print }' data.txt
```

---

## 5. Patterns

Patterns control which records an action applies to. AWK checks the pattern for each record; if it's true (or matches), the action runs.

### 5.1 No Pattern (Match Everything)

```awk
{ print $1 }
```

Runs on every line. Prints the first field of every record.

### 5.2 Comparison Patterns

Use standard comparison operators to match records based on field values:

```awk
$2 > 100 { print }           # Lines where field 2 is greater than 100
$3 == "Engineer" { print }   # Lines where field 3 equals "Engineer"
NF > 3 { print }             # Lines with more than 3 fields
NR == 5 { print }            # Only line number 5
```

### 5.3 Regular Expression Patterns

Match lines where any part of the record matches a pattern:

```awk
/error/ { print }          # Lines containing "error"
/^Alice/ { print }         # Lines starting with "Alice"
/[0-9]+/ { print }         # Lines containing at least one digit
!/warning/ { print }       # Lines NOT containing "warning"
```

You can also match against a specific field:

```awk
$3 ~ /Engineer/ { print }   # Field 3 matches "Engineer"
$3 !~ /Manager/ { print }   # Field 3 does NOT match "Manager"
```

The `~` operator means "matches" and `!~` means "does not match."

### 5.4 Compound Patterns

Combine patterns with `&&` (AND) and `||` (OR):

```awk
$2 > 25 && $3 == "Engineer" { print }   # Both conditions must be true
/error/ || /warning/ { print }           # Either condition can be true
```

### 5.5 Range Patterns

Process a range of lines between two patterns:

```awk
/START/,/END/ { print }
```

This prints every line from the first line matching `START` through the next line matching `END`, inclusive.

---

## 6. Actions

Actions are blocks of code enclosed in `{ }` that execute when the pattern matches. They can contain print statements, calculations, conditionals, loops, and more.

### 6.1 print

The workhorse of AWK output:

```awk
{ print }           # Print the whole line ($0)
{ print $1 }        # Print field 1
{ print $1, $3 }    # Print fields 1 and 3, separated by OFS
{ print $1 " " $3 } # Print fields 1 and 3, joined by a space (string concatenation)
{ print NR, $0 }    # Print line number, then the whole line
```

The comma in `print` uses the **output field separator** (OFS, default is a space). String concatenation (no comma) directly glues strings together.

### 6.2 printf

For formatted output, use `printf` (just like C):

```awk
{ printf "%-15s %5d\n", $1, $2 }
```

Format specifiers:

| Specifier | Meaning |
|-----------|---------|
| `%s`      | String  |
| `%d`      | Integer |
| `%f`      | Float   |
| `%e`      | Scientific notation |
| `%.2f`    | Float with 2 decimal places |
| `%-10s`   | Left-aligned string in 10-character field |
| `%10s`    | Right-aligned string in 10-character field |

```awk
{ printf "Name: %-10s Age: %3d\n", $1, $2 }
```

---

## 7. Built-in Variables

AWK provides several built-in variables that let you control behavior and access metadata:

### Record and Field Variables

| Variable | Meaning |
|----------|---------|
| `$0`     | The entire current record (line) |
| `$1`–`$n` | Individual fields |
| `NF`     | Number of fields in the current record |
| `NR`     | Number of records (lines) read so far, across all files |
| `FNR`    | Number of records read from the current file only |

### Separator Variables

| Variable | Meaning | Default |
|----------|---------|---------|
| `FS`     | Field separator (input) | Space/tab |
| `OFS`    | Output field separator | Space |
| `RS`     | Record separator (input) | Newline |
| `ORS`    | Output record separator | Newline |

### Other Variables

| Variable | Meaning |
|----------|---------|
| `FILENAME` | Name of the current input file |
| `ARGC`   | Number of command-line arguments |
| `ARGV`   | Array of command-line arguments |

### Changing the Field Separator

This is extremely common. Use `-F` on the command line, or set `FS` in a `BEGIN` block:

```bash
# Process a CSV file
awk -F',' '{ print $1, $3 }' data.csv

# Process a colon-delimited file like /etc/passwd
awk -F':' '{ print $1, $3 }' /etc/passwd

# Use a tab as separator
awk -F'\t' '{ print $2 }' data.tsv

# Multi-character separator
awk -F' :: ' '{ print $1 }' data.txt

# Set FS in BEGIN block
awk 'BEGIN { FS = "," } { print $1 }' data.csv
```

### Changing the Output Field Separator

```awk
BEGIN { OFS = "," }
{ print $1, $2, $3 }      # Outputs: field1,field2,field3
```

A useful trick: assigning any field to itself with a custom OFS rebuilds `$0` with the new separator:

```awk
BEGIN { FS = ","; OFS = "|" }
{ $1 = $1; print }         # Converts CSV to pipe-delimited
```

---

## 8. Operators

AWK supports arithmetic, comparison, logical, and string operators.

### Arithmetic Operators

```awk
$2 + $3    # Addition
$2 - $3    # Subtraction
$2 * $3    # Multiplication
$2 / $3    # Division
$2 % $3    # Modulo (remainder)
$2 ^ $3    # Exponentiation (or **)
```

### Arithmetic Assignment Operators

```awk
x = 5
x += 3    # x is now 8
x -= 2    # x is now 6
x *= 4    # x is now 24
x /= 6    # x is now 4
x++       # x is now 5 (increment)
x--       # x is now 4 (decrement)
```

### Comparison Operators

```awk
$2 == 100   # Equal to
$2 != 100   # Not equal to
$2 > 100    # Greater than
$2 < 100    # Less than
$2 >= 100   # Greater than or equal to
$2 <= 100   # Less than or equal to
```

**Important:** AWK automatically treats values as numbers or strings based on context. `"10" > "9"` is false numerically but true as strings (because "1" < "9" lexicographically). Be explicit: add 0 to force numeric comparison (`$1 + 0 > 9`).

### String Concatenation

AWK uses a space (or nothing) between values to concatenate strings:

```awk
{ full_name = $1 " " $2; print full_name }
{ full_name = $1 $2; print full_name }     # No space between
```

### Regular Expression Operators

```awk
$0 ~ /pattern/    # True if $0 matches
$0 !~ /pattern/   # True if $0 does not match
```

---

## 9. Control Flow

### 9.1 if / else if / else

```awk
{
    if ($2 > 90) {
        print $1, "A"
    } else if ($2 > 80) {
        print $1, "B"
    } else if ($2 > 70) {
        print $1, "C"
    } else {
        print $1, "F"
    }
}
```

### 9.2 while Loop

```awk
{
    i = 1
    while (i <= NF) {
        print i, $i
        i++
    }
}
```

### 9.3 for Loop

```awk
# C-style for loop
{
    for (i = 1; i <= NF; i++) {
        print "Field", i, ":", $i
    }
}

# for-in loop (iterates over array keys)
{
    for (key in myarray) {
        print key, myarray[key]
    }
}
```

### 9.4 do-while Loop

```awk
{
    i = 1
    do {
        print $i
        i++
    } while (i <= NF)
}
```

### 9.5 Loop Control: next, break, continue, exit

```awk
/^#/ { next }         # Skip comment lines; go to next record
{ 
    for (i = 1; i <= 10; i++) {
        if (i == 5) break      # Exit the loop entirely
        if (i % 2 == 0) continue  # Skip even numbers
        print i
    }
}
END_PATTERN { exit }  # Stop processing entirely; jumps to END block
```

The `next` statement is one of the most useful tools in AWK. It tells AWK to stop processing the current record and move on to the next one. It's often used to skip lines you don't want to process:

```awk
NR == 1 { next }        # Skip the header row
/^$/ { next }           # Skip blank lines
$3 == "" { next }       # Skip lines where field 3 is empty
```

---

## 10. Arrays

AWK supports associative arrays (also called dictionaries or hash maps). Unlike most languages, you don't need to declare them — just use them. Array keys can be strings or numbers.

### Basic Array Usage

```awk
# Count occurrences of each value in field 1
{ count[$1]++ }

END {
    for (word in count) {
        print word, count[word]
    }
}
```

### Checking if a Key Exists

```awk
if ("Alice" in myarray) { print "Found Alice" }
if (!("Bob" in myarray)) { print "No Bob" }
```

### Deleting Array Elements

```awk
delete myarray["key"]    # Delete a specific element
delete myarray           # Delete the entire array
```

### Multi-Dimensional Arrays

AWK simulates multi-dimensional arrays by concatenating keys with a separator (SUBSEP, which is `\034` by default):

```awk
{ sales[$1][$2] += $3 }        # This won't work as-is
{ sales[$1, $2] += $3 }        # Use comma — AWK joins with SUBSEP
END {
    for (key in sales) {
        split(key, parts, SUBSEP)
        print parts[1], parts[2], sales[key]
    }
}
```

### Example: Summing by Category

```
# Input: product.txt
Apple    fruit    1.50
Banana   fruit    0.75
Carrot   veggie   0.50
Apple    fruit    2.00
Broccoli veggie   1.25
```

```awk
{ total[$2] += $3; count[$2]++ }
END {
    for (cat in total) {
        printf "%-10s Total: $%.2f  Count: %d\n", cat, total[cat], count[cat]
    }
}
```

---

## 11. Built-in Functions

AWK has a rich set of built-in functions for string manipulation, math, and I/O.

### 11.1 String Functions

**`length(string)`** — Returns the length of a string:
```awk
{ print length($0), $0 }          # Print line length before each line
{ if (length($1) > 10) print }    # Print lines where field 1 is long
```

**`substr(string, start, length)`** — Extract a substring (1-indexed):
```awk
{ print substr($1, 1, 3) }        # First 3 characters of field 1
{ print substr($0, 5) }           # Everything from position 5 onward
```

**`index(string, target)`** — Find position of target in string (0 if not found):
```awk
{ if (index($0, "error") > 0) print }
```

**`split(string, array, separator)`** — Split a string into an array:
```awk
{
    n = split($0, parts, ":")
    for (i = 1; i <= n; i++) print parts[i]
}
```

**`sub(regex, replacement, string)`** — Replace the first match:
```awk
{ sub(/error/, "ERROR", $0); print }   # Replace first "error" with "ERROR"
{ sub(/^[ \t]+/, "", $0); print }      # Trim leading whitespace
```

**`gsub(regex, replacement, string)`** — Replace all matches:
```awk
{ gsub(/,/, "\t", $0); print }         # Convert CSV to TSV
{ gsub(/ +/, " ", $0); print }         # Collapse multiple spaces
{ n = gsub(/error/, "ERROR"); print n, $0 }  # gsub returns count of replacements
```

**`match(string, regex)`** — Test if string matches regex; sets `RSTART` and `RLENGTH`:
```awk
{
    if (match($0, /[0-9]+/)) {
        print "Found number at position", RSTART, "length", RLENGTH
        print substr($0, RSTART, RLENGTH)   # Extract the match
    }
}
```

**`sprintf(format, ...)`** — Format a string like printf, but return it instead of printing:
```awk
{ padded = sprintf("%-20s", $1); print padded, $2 }
```

**`tolower(string)` and `toupper(string)`**:
```awk
{ print tolower($0) }
{ print toupper($1), $2 }
```

### 11.2 Math Functions

```awk
{ print sqrt($1) }         # Square root
{ print int($1) }          # Truncate to integer (floor for positive numbers)
{ print log($1) }          # Natural logarithm
{ print exp($1) }          # e raised to the power
{ print sin($1) }          # Sine (radians)
{ print cos($1) }          # Cosine (radians)
{ print atan2($1, $2) }    # Arctangent of $1/$2
{ print $1 % $2 }          # Modulo

# Random numbers
{ print rand() }           # Random float between 0 and 1
BEGIN { srand() }          # Seed random number generator with current time
{ print int(rand() * 100) } # Random integer 0-99
```

### 11.3 I/O Functions

**`getline`** — Read the next line from input (advanced):
```awk
{ 
    getline line < "other_file.txt"
    print line
}
```

**`print > "file"`** — Write to a file:
```awk
{ print $0 > "output.txt" }       # Overwrite (opened once, appended thereafter in same run)
{ print $0 >> "output.txt" }      # Always append
```

**`print | "command"`** — Pipe output to a command:
```awk
{ print $0 | "sort" }
{ print $0 | "mail -s 'Report' admin@example.com" }
```

**`close("file_or_command")`** — Close a file or pipe (important when writing multiple files):
```awk
{
    print $0 > $1 ".txt"    # Write each record to a file named after field 1
}
END {
    # All files are closed automatically, but you can close explicitly:
    # close("myfile.txt")
}
```

---

## 12. User-Defined Functions

You can define your own functions in AWK for reusable logic:

```awk
function max(a, b) {
    return (a > b) ? a : b
}

function min(a, b) {
    return (a < b) ? a : b
}

function trim(str) {
    sub(/^[ \t]+/, "", str)
    sub(/[ \t]+$/, "", str)
    return str
}

{
    print max($1, $2), min($1, $2), trim($3)
}
```

**Important notes about functions:**

- Functions must be defined at the top level (not inside another rule).
- All variables in AWK are global by default — including inside functions. To make local variables, add extra parameters to the function signature that you never pass arguments for. By convention, these are separated from real parameters with extra spaces:

```awk
function process(input,    local_var, i, tmp) {
    # local_var, i, and tmp are local to this function
    local_var = input * 2
    return local_var
}
```

---

## 13. BEGIN and END Blocks

`BEGIN` and `END` are special patterns that run before any input is read and after all input is processed, respectively.

```awk
BEGIN {
    print "=== Report Start ==="
    FS = ","          # Set field separator before any input is read
    total = 0
}

{
    total += $2
    print $1, $2
}

END {
    print "=== Total:", total, "==="
    print "=== Processed", NR, "records ==="
}
```

**Common uses of BEGIN:**
- Set `FS`, `OFS`, `RS`, `ORS`
- Print headers
- Initialize variables

**Common uses of END:**
- Print summaries, totals, averages
- Print footers
- Write final output

You can have multiple `BEGIN` and `END` blocks — they all execute in order.

---

## 14. Working with Multiple Files

When AWK processes multiple files, `NR` keeps counting across files while `FNR` resets for each file:

```bash
awk '{ print FILENAME, FNR, NR, $0 }' file1.txt file2.txt
```

A classic pattern: use the first file as a lookup table and the second file as data:

```awk
# Load a lookup table from the first file
FNR == NR {
    lookup[$1] = $2
    next
}

# Process the second file using the lookup table
{
    if ($1 in lookup) {
        print $0, lookup[$1]
    }
}
```

Run as: `awk -f program.awk lookup.txt data.txt`

The trick here is `FNR == NR` — this condition is only true while reading the first file (since both counters are equal for the first file). When the second file starts, `FNR` resets to 1 but `NR` continues, so the condition becomes false.

---

## 15. AWK One-Liners Cookbook

These practical one-liners cover the most common everyday tasks.

### Text Extraction

```bash
# Print the 3rd field of every line
awk '{ print $3 }' file.txt

# Print the last field of every line
awk '{ print $NF }' file.txt

# Print fields 2 through 4
awk '{ print $2, $3, $4 }' file.txt

# Print all fields except the first (fields 2 to NF)
awk '{ $1=""; print }' file.txt

# Print the first 3 fields in reverse order
awk '{ print $3, $2, $1 }' file.txt

# Print specific lines (e.g., lines 5 to 10)
awk 'NR>=5 && NR<=10' file.txt

# Print every other line
awk 'NR%2==0' file.txt

# Print the last line
awk 'END { print }' file.txt

# Print the nth line
awk 'NR==42' file.txt
```

### Filtering

```bash
# Print lines containing a pattern
awk '/pattern/' file.txt

# Print lines NOT containing a pattern
awk '!/pattern/' file.txt

# Print lines where field 2 is greater than 100
awk '$2 > 100' file.txt

# Print lines where field 3 equals "active"
awk '$3 == "active"' file.txt

# Print lines longer than 80 characters
awk 'length > 80' file.txt

# Print lines with more than 5 fields
awk 'NF > 5' file.txt

# Skip blank lines
awk 'NF > 0' file.txt

# Skip comment lines and blank lines
awk '!/^#/ && NF > 0' file.txt
```

### Calculations and Aggregation

```bash
# Sum all values in field 1
awk '{ sum += $1 } END { print sum }' file.txt

# Average of field 2
awk '{ sum += $2 } END { print sum/NR }' file.txt

# Count lines matching a pattern
awk '/error/ { count++ } END { print count }' file.txt

# Find maximum value in field 1
awk 'NR==1 || $1>max { max=$1 } END { print max }' file.txt

# Find minimum value in field 1
awk 'NR==1 || $1<min { min=$1 } END { print min }' file.txt

# Count unique values in field 1
awk '!seen[$1]++' file.txt | wc -l

# Print duplicate lines only
awk 'seen[$0]++' file.txt

# Remove duplicate lines (preserve order)
awk '!seen[$0]++' file.txt
```

### Text Transformation

```bash
# Add line numbers
awk '{ print NR, $0 }' file.txt

# Swap fields 1 and 2
awk '{ t=$1; $1=$2; $2=t; print }' file.txt

# Convert spaces to tabs
awk 'BEGIN{OFS="\t"} { $1=$1; print }' file.txt

# Convert CSV to TSV
awk 'BEGIN{FS=","; OFS="\t"} { $1=$1; print }' file.csv

# Add a header line
awk 'BEGIN { print "Name,Age,Role" } { print }' file.txt

# Remove the header line
awk 'NR>1' file.txt

# Double-space a file (add blank line after each line)
awk '{ print; print "" }' file.txt

# Convert to uppercase
awk '{ print toupper($0) }' file.txt

# Trim leading/trailing whitespace from each line
awk '{ gsub(/^[ \t]+|[ \t]+$/, ""); print }' file.txt
```

### Log File Analysis

```bash
# Count errors per hour from a timestamp log (assuming HH:MM:SS format in field 1)
awk '/error/ { hour=substr($1,1,2); count[hour]++ }
     END { for (h in count) print h":00", count[h] | "sort" }' app.log

# Find the top 10 most common IP addresses
awk '{ count[$1]++ }
     END { for (ip in count) print count[ip], ip }' access.log | sort -rn | head -10

# Calculate total bytes transferred (field 10 in Apache logs)
awk '{ if ($10 ~ /^[0-9]+$/) bytes += $10 }
     END { printf "Total: %.2f MB\n", bytes/1048576 }' access.log
```

---

## 16. Real-World Examples

### Example 1: Process /etc/passwd

```bash
# Print username and home directory
awk -F':' '{ print $1, $6 }' /etc/passwd

# Find users with /bin/bash as their shell
awk -F':' '$7 == "/bin/bash" { print $1 }' /etc/passwd

# Count users by shell type
awk -F':' '{ shells[$7]++ }
           END { for (s in shells) print shells[s], s }' /etc/passwd | sort -rn
```

### Example 2: Summarize a Sales Report

Given a file `sales.csv`:
```
Product,Region,Salesperson,Amount
Widget A,North,Alice,1200
Widget B,South,Bob,850
Widget A,East,Carol,950
Widget B,North,Alice,1100
Widget A,South,Bob,760
```

```awk
BEGIN { FS="," }
NR==1 { next }   # Skip header

{
    product_total[$1] += $4
    region_total[$2] += $4
    person_total[$3] += $4
    grand_total += $4
}

END {
    print "\n=== Sales by Product ==="
    for (p in product_total) printf "  %-12s $%8.2f\n", p, product_total[p]
    
    print "\n=== Sales by Region ==="
    for (r in region_total) printf "  %-12s $%8.2f\n", r, region_total[r]
    
    print "\n=== Sales by Person ==="
    for (s in person_total) printf "  %-12s $%8.2f\n", s, person_total[s]
    
    printf "\n=== Grand Total: $%.2f ===\n", grand_total
}
```

Run with: `awk -f report.awk sales.csv`

### Example 3: Monitor a Log File

Parse an Apache/Nginx access log and produce a summary:

```awk
{
    # Count by status code (field 9 in common log format)
    status[$9]++
    
    # Count by hour (field 4 looks like: [10/Feb/2024:14:23:45)
    split($4, t, ":")
    hour = substr(t[2], 1)
    requests_per_hour[hour]++
    
    # Track 404s
    if ($9 == 404) {
        not_found[$7]++   # field 7 is the URL
    }
    
    total++
}

END {
    print "=== HTTP Status Codes ==="
    for (s in status) printf "  %s: %d (%.1f%%)\n", s, status[s], 100*status[s]/total
    
    print "\n=== Top 5 Missing Pages (404) ==="
    for (url in not_found) print not_found[url], url | "sort -rn | head -5"
}
```

### Example 4: Data Validation

Check a CSV data file for common problems:

```awk
BEGIN {
    FS = ","
    errors = 0
}

NR == 1 { next }  # Skip header

{
    # Check field count
    if (NF != 4) {
        printf "Line %d: Expected 4 fields, got %d\n", NR, NF
        errors++
    }
    
    # Check that field 3 (age) is a positive number
    if ($3 !~ /^[0-9]+$/ || $3 + 0 <= 0) {
        printf "Line %d: Invalid age '%s'\n", NR, $3
        errors++
    }
    
    # Check that field 1 (email) has @ sign
    if ($1 !~ /@/) {
        printf "Line %d: Invalid email '%s'\n", NR, $1
        errors++
    }
    
    # Check for empty required fields
    for (i = 1; i <= 4; i++) {
        if ($i == "") {
            printf "Line %d: Field %d is empty\n", NR, i
            errors++
        }
    }
}

END {
    if (errors == 0) print "Validation passed: no errors found."
    else printf "Validation failed: %d error(s) found.\n", errors
}
```

### Example 5: Reformat Data

Convert a pipe-delimited file to a nicely formatted report:

```awk
BEGIN {
    FS = "|"
    printf "%-20s %-15s %-10s %10s\n", "Name", "Department", "Title", "Salary"
    printf "%s\n", "------------------------------------------------------------"
}

NR > 1 {
    printf "%-20s %-15s %-10s %10s\n", $1, $2, $3, $4
    total += $4
    count++
}

END {
    printf "%s\n", "------------------------------------------------------------"
    printf "%-20s %-15s %-10s %10.2f\n", "AVERAGE", "", "", total/count
}
```

---

## 17. Tips, Tricks, and Gotchas

### Gotcha 1: Numeric vs. String Comparison

AWK decides whether to compare numerically or as strings based on context. If both operands look like numbers, it compares numerically; otherwise, it compares as strings.

```awk
# This might surprise you:
"10" > "9"    # FALSE as strings ("1" < "9" alphabetically)
10 > 9        # TRUE as numbers

# Force numeric comparison:
($1 + 0) > ($2 + 0)

# Force string comparison:
($1 "") > ($2 "")
```

### Gotcha 2: Uninitialized Variables

In AWK, uninitialized variables are 0 (numeric) or "" (string) — this is actually a feature. It means you can write `count++` without ever initializing `count = 0`, and it works fine.

### Gotcha 3: print vs. printf Newlines

`print` automatically adds a newline (ORS). `printf` does not — you must include `\n` explicitly.

```awk
{ print "Hello" }        # Adds newline automatically
{ printf "Hello\n" }     # Must add \n yourself
{ printf "Hello" }       # No newline — next output continues on same line
```

### Gotcha 4: Regex Anchors in Field Matching

```awk
$1 ~ /^error/    # Field 1 STARTS with "error"
$1 ~ /error$/    # Field 1 ENDS with "error"
$1 ~ /^error$/   # Field 1 IS exactly "error"
$1 == "error"    # Also exactly "error" — usually clearer for exact matches
```

### Gotcha 5: Modifying $0

When you change a field like `$2 = "new_value"`, AWK rebuilds `$0` using OFS as the separator. This can change your output format unexpectedly if OFS is not what you expect.

```awk
BEGIN { OFS = "," }
{ $2 = toupper($2); print }   # $0 is rebuilt with commas between fields
```

### Gotcha 6: The `next` Statement

Forgetting to use `next` when you need it is a common mistake. If you have a `BEGIN`-style action for the header row, always use `next` to prevent the rest of your rules from running on it too:

```awk
NR == 1 { print "Header:", $0; next }   # next skips remaining rules for this record
{ process($0) }
```

### Tip: Use -v to Pass Shell Variables

Never try to embed shell variables inside single-quoted AWK programs. Instead, use `-v`:

```bash
# WRONG:
threshold=100
awk '{ if ($2 > $threshold) print }' file.txt   # $threshold is AWK variable, not shell!

# RIGHT:
awk -v threshold="$threshold" '{ if ($2 > threshold) print }' file.txt
```

### Tip: Suppress Output with 0 or 1 as Action

AWK is truthy: any non-zero, non-empty value is true. You can use this for compact filtering:

```awk
$2 > 100        # Prints matching lines (implicit { print $0 })
!/^#/           # Prints non-comment lines
NR%2            # Prints odd-numbered lines (NR%2 is 1 for odd, 0 for even)
```

### Tip: Pipe to sort in END

You can open a pipe to sort inside AWK and it stays open for all your print statements in that block:

```awk
END {
    for (key in counts) {
        print counts[key], key | "sort -rn"
    }
}
```

AWK keeps the pipe open and passes all output through sort at once.

### Tip: gawk Extensions

If you have `gawk` available, you get extra features:
- `gensub(regex, replacement, how, string)` — more powerful substitution with backreferences
- `PROCINFO` array — information about the running process
- `strftime()` and `mktime()` — time formatting functions
- `patsplit()` — split by regex
- `BEGINFILE` and `ENDFILE` — run code at the start/end of each input file

```bash
# Use gawk's gensub for backreferences
echo "hello world" | gawk '{ print gensub(/(\w+) (\w+)/, "\\2 \\1", "g") }'
# Output: world hello
```

---

## Quick Reference Card

```
STRUCTURE:       awk 'BEGIN{} pattern{action} END{}' file

FIELD VARS:      $0=whole line, $1..$n=fields, NF=field count, NR=record number

SEPARATORS:      FS=input field sep (-F on cmdline), OFS=output field sep
                 RS=input record sep, ORS=output record sep

PATTERNS:        /regex/, $2>100, NR==5, /start/,/end/, !pattern

PRINT:           print $1,$2    (uses OFS)
                 print $1 $2    (concatenation, no separator)
                 printf "%-10s %d\n", $1, $2

STRING FNS:      length(s), substr(s,pos,len), index(s,t), split(s,a,sep)
                 sub(/re/,repl,s), gsub(/re/,repl,s), match(s,/re/)
                 toupper(s), tolower(s), sprintf(fmt,...)

MATH FNS:        sqrt, int, log, exp, sin, cos, atan2, rand, srand

ARRAYS:          a[key]=val, key in a, delete a[key], for(k in a)

CONTROL:         if/else, while, for, do-while, next, break, continue, exit

FUNCTIONS:       function name(params,    local_vars) { ... return val }
```

---

*This guide was written for AWK beginners and covers POSIX-standard AWK with notes on GNU AWK (gawk) extensions. For the definitive reference, see the book "The AWK Programming Language" by Aho, Weinberger, and Kernighan, or run `man awk` on your system.*
