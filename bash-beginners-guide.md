# 🐚 Bash: A Comprehensive Beginner's Guide

[![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-nc-sa/4.0/)


Bash (Bourne Again SHell) is the default command-line shell on most Linux distributions and macOS. It's a powerful tool for automating tasks, managing files, and controlling your system. This guide will take you from absolute beginner to confidently writing your own Bash scripts.


## Table of Contents

- [1. What is Bash?](#1-what-is-bash)
- [2. Opening a Terminal](#2-opening-a-terminal)
- [3. Navigating the File System](#3-navigating-the-file-system)
  - [Essential Navigation Commands](#essential-navigation-commands)
  - [`ls` Options](#ls-options)
  - [Understanding Paths](#understanding-paths)
- [4. Working with Files and Directories](#4-working-with-files-and-directories)
  - [Creating Files and Directories](#creating-files-and-directories)
  - [Copying, Moving, and Renaming](#copying-moving-and-renaming)
  - [Deleting Files and Directories](#deleting-files-and-directories)
  - [Finding Files](#finding-files)
- [5. Viewing and Editing Files](#5-viewing-and-editing-files)
  - [Viewing File Contents](#viewing-file-contents)
  - [Searching Within Files](#searching-within-files)
  - [Text Editors in the Terminal](#text-editors-in-the-terminal)
  - [Word Count](#word-count)
- [6. Permissions](#6-permissions)
  - [Reading Permissions](#reading-permissions)
  - [Changing Permissions with `chmod`](#changing-permissions-with-chmod)
  - [Changing Ownership](#changing-ownership)
- [7. Redirection and Pipes](#7-redirection-and-pipes)
  - [Standard Streams](#standard-streams)
  - [Redirection Operators](#redirection-operators)
  - [Pipes](#pipes)
- [8. Variables](#8-variables)
  - [Defining and Using Variables](#defining-and-using-variables)
  - [Variable Rules](#variable-rules)
  - [Command Substitution](#command-substitution)
  - [Special Variables](#special-variables)
- [9. User Input](#9-user-input)
- [10. Conditionals](#10-conditionals)
  - [The `if` Statement](#the-if-statement)
  - [File Test Operators](#file-test-operators)
  - [Comparison Operators](#comparison-operators)
  - [Logical Operators](#logical-operators)
  - [The `case` Statement](#the-case-statement)
- [11. Loops](#11-loops)
  - [`for` Loop](#for-loop)
  - [`while` Loop](#while-loop)
  - [`until` Loop](#until-loop)
  - [Loop Control](#loop-control)
- [12. Functions](#12-functions)
  - [Functions with Return Values](#functions-with-return-values)
  - [Local Variables](#local-variables)
- [13. Arrays](#13-arrays)
  - [Associative Arrays (Dictionaries)](#associative-arrays-dictionaries)
- [14. String Manipulation](#14-string-manipulation)
- [15. Arithmetic](#15-arithmetic)
- [16. Script Best Practices](#16-script-best-practices)
  - [The Shebang Line](#the-shebang-line)
  - [A Well-Structured Script](#a-well-structured-script)
  - [Strict Mode](#strict-mode)
  - [Making a Script Executable](#making-a-script-executable)
  - [Quoting](#quoting)
  - [Error Handling](#error-handling)
- [17. Useful Built-in Commands](#17-useful-built-in-commands)
- [18. Process Management](#18-process-management)
- [19. Environment Variables](#19-environment-variables)
- [20. Tips and Tricks](#20-tips-and-tricks)
  - [Keyboard Shortcuts](#keyboard-shortcuts)
  - [Brace Expansion](#brace-expansion)
  - [Here Documents](#here-documents)
  - [The `xargs` Command](#the-xargs-command)
  - [Useful One-Liners](#useful-one-liners)
- [Next Steps](#next-steps)

---
## 1. What is Bash?

Bash is both a **command-line interpreter** (you type commands and it runs them) and a **scripting language** (you can write programs called "scripts" that Bash executes). It was written as a free replacement for the original Bourne Shell (`sh`) and has been the standard shell on Linux since the late 1980s.

A Bash **script** is simply a plain text file containing a series of commands. When you run the script, Bash executes each command in order, top to bottom.

---


[↑ Goto TOC](#table-of-contents)

## 2. Opening a Terminal

- **Linux:** Search for "Terminal" in your applications menu, or press `Ctrl + Alt + T`.
- **macOS:** Open `Applications > Utilities > Terminal`, or press `Cmd + Space` and type "Terminal".
- **Windows:** Use [WSL (Windows Subsystem for Linux)](https://learn.microsoft.com/en-us/windows/wsl/install) or Git Bash.

When you open a terminal, you'll see a **prompt** — something like:

```
username@hostname:~$
```

The `$` means you're a regular user. A `#` means you're the root (administrator) user.

---


[↑ Goto TOC](#table-of-contents)

## 3. Navigating the File System


[↑ Goto TOC](#table-of-contents)

### Essential Navigation Commands

| Command | Description |
|---|---|
| `pwd` | Print Working Directory — shows where you are |
| `ls` | List files and folders in the current directory |
| `cd <dir>` | Change Directory |
| `cd ..` | Go up one level |
| `cd ~` | Go to your home directory |
| `cd -` | Go to the previous directory |


[↑ Goto TOC](#table-of-contents)

### `ls` Options

```bash
ls -l       # Long format (permissions, owner, size, date)
ls -a       # Show hidden files (files starting with .)
ls -lh      # Long format with human-readable file sizes
ls -lt      # Sort by modification time (newest first)
ls -R       # Recursively list all subdirectories
```


[↑ Goto TOC](#table-of-contents)

### Understanding Paths

- **Absolute path:** Starts from root `/`. Example: `/home/user/documents`
- **Relative path:** Relative to your current location. Example: `./documents` or just `documents`

```bash
cd /etc             # Navigate to /etc using an absolute path
cd documents        # Navigate into 'documents' relative to current dir
cd ../../other      # Go up two levels, then into 'other'
```

---


[↑ Goto TOC](#table-of-contents)

## 4. Working with Files and Directories


[↑ Goto TOC](#table-of-contents)

### Creating Files and Directories

```bash
touch filename.txt          # Create an empty file (or update its timestamp)
mkdir my_folder             # Create a directory
mkdir -p a/b/c              # Create nested directories in one go
```


[↑ Goto TOC](#table-of-contents)

### Copying, Moving, and Renaming

```bash
cp file.txt copy.txt        # Copy a file
cp -r folder/ backup/       # Copy a directory recursively
mv file.txt newname.txt     # Rename a file
mv file.txt /tmp/           # Move a file to /tmp/
mv folder/ /tmp/            # Move a directory
```


[↑ Goto TOC](#table-of-contents)

### Deleting Files and Directories

```bash
rm file.txt                 # Delete a file
rm -i file.txt              # Ask for confirmation before deleting
rm -r folder/               # Delete a directory and all its contents
rm -rf folder/              # Force delete — NO confirmation (use with caution!)
rmdir empty_folder          # Delete an empty directory
```

> ⚠️ **Warning:** There is no Recycle Bin in Bash. Deleted files are gone immediately. Use `rm -i` when you're unsure.


[↑ Goto TOC](#table-of-contents)

### Finding Files

```bash
find . -name "file.txt"             # Find by name in current directory
find /home -name "*.log"            # Find all .log files in /home
find . -type d                      # Find only directories
find . -type f -mtime -7            # Files modified in the last 7 days
find . -size +10M                   # Files larger than 10 MB
```

---


[↑ Goto TOC](#table-of-contents)

## 5. Viewing and Editing Files


[↑ Goto TOC](#table-of-contents)

### Viewing File Contents

```bash
cat file.txt                # Print entire file to the terminal
less file.txt               # View file page by page (press q to quit)
head file.txt               # Show first 10 lines
head -n 20 file.txt         # Show first 20 lines
tail file.txt               # Show last 10 lines
tail -f logfile.log         # Follow a file in real time (great for logs)
```


[↑ Goto TOC](#table-of-contents)

### Searching Within Files

```bash
grep "word" file.txt                # Find lines containing "word"
grep -i "word" file.txt             # Case-insensitive search
grep -r "word" ./folder/            # Recursive search in a directory
grep -n "word" file.txt             # Show line numbers
grep -v "word" file.txt             # Show lines that do NOT contain "word"
grep -c "word" file.txt             # Count matching lines
```


[↑ Goto TOC](#table-of-contents)

### Text Editors in the Terminal

```bash
nano file.txt       # Simple, beginner-friendly editor
vim file.txt        # Powerful but has a steep learning curve
```

In `nano`: Use `Ctrl+O` to save, `Ctrl+X` to exit.


[↑ Goto TOC](#table-of-contents)

### Word Count

```bash
wc file.txt         # Lines, words, and characters
wc -l file.txt      # Count lines only
wc -w file.txt      # Count words only
```

---


[↑ Goto TOC](#table-of-contents)

## 6. Permissions

Every file and directory has permissions that control who can read, write, or execute it.


[↑ Goto TOC](#table-of-contents)

### Reading Permissions

Run `ls -l` to see permissions:

```
-rwxr-xr-- 1 alice staff 1234 Jan 1 12:00 script.sh
```

Breaking this down:

```
- rwx r-x r--
│ │   │   └── Other: read only
│ │   └────── Group: read and execute
│ └────────── Owner: read, write, execute
└──────────── File type (- = file, d = directory, l = symlink)
```


[↑ Goto TOC](#table-of-contents)

### Changing Permissions with `chmod`

```bash
chmod +x script.sh          # Make a file executable
chmod -w file.txt           # Remove write permission
chmod 755 script.sh         # Set permissions using octal notation
chmod 644 file.txt          # Owner can read/write; others can read
chmod -R 755 folder/        # Apply permissions recursively
```

**Octal notation quick reference:**

| Number | Permission |
|---|---|
| 7 | Read + Write + Execute (rwx) |
| 6 | Read + Write (rw-) |
| 5 | Read + Execute (r-x) |
| 4 | Read only (r--) |
| 0 | No permission (---) |


[↑ Goto TOC](#table-of-contents)

### Changing Ownership

```bash
chown alice file.txt            # Change file owner to alice
chown alice:staff file.txt      # Change owner and group
sudo chown -R alice folder/     # Recursively change ownership
```

---


[↑ Goto TOC](#table-of-contents)

## 7. Redirection and Pipes

One of Bash's most powerful features is the ability to redirect output and chain commands together.


[↑ Goto TOC](#table-of-contents)

### Standard Streams

- **stdin (0):** Input (usually keyboard)
- **stdout (1):** Standard output (usually screen)
- **stderr (2):** Error output (usually screen)


[↑ Goto TOC](#table-of-contents)

### Redirection Operators

```bash
command > file.txt          # Redirect stdout to a file (overwrites)
command >> file.txt         # Append stdout to a file
command 2> error.log        # Redirect stderr to a file
command 2>&1                # Redirect stderr to same place as stdout
command > out.txt 2>&1      # Redirect both stdout and stderr to a file
command < input.txt         # Use a file as stdin
```


[↑ Goto TOC](#table-of-contents)

### Pipes

The pipe `|` sends the output of one command as the input to the next:

```bash
ls -l | grep ".txt"             # List files, then filter for .txt
cat file.txt | wc -l            # Count lines in a file
ps aux | grep "firefox"         # Find Firefox processes
cat access.log | sort | uniq    # Sort and remove duplicates
history | tail -20              # Show last 20 commands
```

---


[↑ Goto TOC](#table-of-contents)

## 8. Variables


[↑ Goto TOC](#table-of-contents)

### Defining and Using Variables

```bash
name="Alice"            # No spaces around =
echo $name              # Access the variable with $
echo "Hello, $name!"    # Use inside double quotes
echo 'Hello, $name!'    # Single quotes — literal, NOT expanded
```


[↑ Goto TOC](#table-of-contents)

### Variable Rules

- Variable names are case-sensitive. `Name` and `name` are different.
- No spaces around `=` when assigning.
- Use `$` when reading a variable, but not when assigning.


[↑ Goto TOC](#table-of-contents)

### Command Substitution

Capture the output of a command into a variable:

```bash
today=$(date)
echo "Today is: $today"

files=$(ls | wc -l)
echo "There are $files files here."
```


[↑ Goto TOC](#table-of-contents)

### Special Variables

| Variable | Meaning |
|---|---|
| `$0` | Name of the script |
| `$1`, `$2`, ... | Positional arguments passed to the script |
| `$#` | Number of arguments |
| `$@` | All arguments as separate words |
| `$*` | All arguments as a single string |
| `$?` | Exit status of the last command (0 = success) |
| `$$` | Process ID of the current shell |
| `$!` | Process ID of the last background command |

---


[↑ Goto TOC](#table-of-contents)

## 9. User Input

```bash
echo "What is your name?"
read name
echo "Hello, $name!"

# Read with a prompt on the same line
read -p "Enter your age: " age
echo "You are $age years old."

# Read a password silently (no echo)
read -sp "Enter password: " password
echo
echo "Password received."
```

---


[↑ Goto TOC](#table-of-contents)

## 10. Conditionals


[↑ Goto TOC](#table-of-contents)

### The `if` Statement

```bash
if [ condition ]; then
    # commands
elif [ other_condition ]; then
    # commands
else
    # commands
fi
```


[↑ Goto TOC](#table-of-contents)

### File Test Operators

```bash
if [ -f "file.txt" ]; then echo "file.txt exists and is a file"; fi
if [ -d "folder" ]; then echo "folder is a directory"; fi
if [ -e "path" ]; then echo "path exists"; fi
if [ -r "file" ]; then echo "file is readable"; fi
if [ -w "file" ]; then echo "file is writable"; fi
if [ -x "file" ]; then echo "file is executable"; fi
if [ -z "$var" ]; then echo "variable is empty"; fi
if [ -n "$var" ]; then echo "variable is not empty"; fi
```


[↑ Goto TOC](#table-of-contents)

### Comparison Operators

**Numbers:**

```bash
[ $a -eq $b ]   # Equal
[ $a -ne $b ]   # Not equal
[ $a -lt $b ]   # Less than
[ $a -le $b ]   # Less than or equal
[ $a -gt $b ]   # Greater than
[ $a -ge $b ]   # Greater than or equal
```

**Strings:**

```bash
[ "$a" = "$b" ]     # Equal
[ "$a" != "$b" ]    # Not equal
[ "$a" < "$b" ]     # Less than (alphabetical)
[ -z "$a" ]         # String is empty
[ -n "$a" ]         # String is not empty
```


[↑ Goto TOC](#table-of-contents)

### Logical Operators

```bash
[ condition1 ] && [ condition2 ]    # AND
[ condition1 ] || [ condition2 ]    # OR
! [ condition ]                     # NOT

# Or inside double brackets (preferred):
[[ condition1 && condition2 ]]
[[ condition1 || condition2 ]]
```


[↑ Goto TOC](#table-of-contents)

### The `case` Statement

```bash
read -p "Enter a fruit: " fruit

case $fruit in
    apple)
        echo "You chose apple!"
        ;;
    banana | plantain)
        echo "You chose a banana-type fruit!"
        ;;
    *)
        echo "Unknown fruit."
        ;;
esac
```

---


[↑ Goto TOC](#table-of-contents)

## 11. Loops


[↑ Goto TOC](#table-of-contents)

### `for` Loop

```bash
# Loop over a list
for color in red green blue; do
    echo "Color: $color"
done

# Loop over a range of numbers
for i in {1..5}; do
    echo "Number: $i"
done

# C-style for loop
for ((i=0; i<10; i++)); do
    echo "i = $i"
done

# Loop over files
for file in *.txt; do
    echo "Processing: $file"
done
```


[↑ Goto TOC](#table-of-contents)

### `while` Loop

```bash
count=1
while [ $count -le 5 ]; do
    echo "Count: $count"
    ((count++))
done

# Read a file line by line
while IFS= read -r line; do
    echo "Line: $line"
done < file.txt
```


[↑ Goto TOC](#table-of-contents)

### `until` Loop

Runs until the condition becomes true (the opposite of `while`):

```bash
count=1
until [ $count -gt 5 ]; do
    echo "Count: $count"
    ((count++))
done
```


[↑ Goto TOC](#table-of-contents)

### Loop Control

```bash
break       # Exit the loop immediately
continue    # Skip to the next iteration
```

```bash
for i in {1..10}; do
    if [ $i -eq 5 ]; then
        continue    # Skip 5
    fi
    if [ $i -eq 8 ]; then
        break       # Stop at 8
    fi
    echo $i
done
```

---


[↑ Goto TOC](#table-of-contents)

## 12. Functions

Functions let you group reusable commands together.

```bash
# Define a function
greet() {
    echo "Hello, $1!"    # $1 is the first argument to the function
}

# Call the function
greet "Alice"
greet "Bob"
```


[↑ Goto TOC](#table-of-contents)

### Functions with Return Values

Bash functions can't return arbitrary values — they can only return an exit code (0–255). Use `echo` to output a value and capture it:

```bash
add() {
    local result=$(( $1 + $2 ))
    echo $result
}

sum=$(add 5 10)
echo "5 + 10 = $sum"
```


[↑ Goto TOC](#table-of-contents)

### Local Variables

Use `local` to prevent variables from leaking out of a function:

```bash
my_function() {
    local temp="I'm local"
    echo $temp
}

my_function
echo $temp      # This will be empty — $temp doesn't exist here
```

---


[↑ Goto TOC](#table-of-contents)

## 13. Arrays

```bash
# Define an array
fruits=("apple" "banana" "cherry")

# Access elements (zero-indexed)
echo ${fruits[0]}       # apple
echo ${fruits[1]}       # banana
echo ${fruits[-1]}      # Last element: cherry

# All elements
echo ${fruits[@]}

# Number of elements
echo ${#fruits[@]}

# Add an element
fruits+=("date")

# Slice an array (start at index 1, get 2 elements)
echo ${fruits[@]:1:2}   # banana cherry

# Loop over an array
for fruit in "${fruits[@]}"; do
    echo $fruit
done
```


[↑ Goto TOC](#table-of-contents)

### Associative Arrays (Dictionaries)

```bash
declare -A person
person["name"]="Alice"
person["age"]="30"

echo ${person["name"]}

# Loop over keys and values
for key in "${!person[@]}"; do
    echo "$key: ${person[$key]}"
done
```

---


[↑ Goto TOC](#table-of-contents)

## 14. String Manipulation

```bash
str="Hello, World!"

echo ${#str}            # Length: 13
echo ${str:7}           # Substring from index 7: World!
echo ${str:7:5}         # 5 chars from index 7: World
echo ${str/World/Bash}  # Replace first match: Hello, Bash!
echo ${str//l/L}        # Replace all matches: HeLLo, WorLd!
echo ${str,,}           # Lowercase: hello, world!
echo ${str^^}           # Uppercase: HELLO, WORLD!

# Remove prefix/suffix patterns
file="backup_2024.tar.gz"
echo ${file#backup_}        # Remove shortest prefix match: 2024.tar.gz
echo ${file##*.}            # Remove longest prefix, get extension: gz
echo ${file%.tar.gz}        # Remove suffix: backup_2024
```

---


[↑ Goto TOC](#table-of-contents)

## 15. Arithmetic

```bash
# Using $(( ))
a=10
b=3
echo $(( a + b ))       # 13
echo $(( a - b ))       # 7
echo $(( a * b ))       # 30
echo $(( a / b ))       # 3  (integer division)
echo $(( a % b ))       # 1  (remainder)
echo $(( a ** b ))      # 1000 (exponent)

# Increment/decrement
((a++))
((a--))
((a += 5))

# For floating-point math, use bc
echo "scale=2; 10 / 3" | bc      # 3.33
echo "scale=4; sqrt(2)" | bc -l  # 1.4142
```

---


[↑ Goto TOC](#table-of-contents)

## 16. Script Best Practices


[↑ Goto TOC](#table-of-contents)

### The Shebang Line

Always start your script with a shebang to tell the system which interpreter to use:

```bash
#!/bin/bash
```


[↑ Goto TOC](#table-of-contents)

### A Well-Structured Script

```bash
#!/bin/bash
# =============================================================
# Script Name: my_script.sh
# Description: A brief description of what this script does
# Author: Your Name
# Date: 2024-01-01
# =============================================================

set -euo pipefail   # Strict mode (see below)

# --- Variables ---
LOG_FILE="/tmp/my_script.log"
NAME="${1:-default}"    # Use first arg, or "default" if not given

# --- Functions ---
log() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $*" | tee -a "$LOG_FILE"
}

main() {
    log "Script started"
    echo "Hello, $NAME!"
    log "Script finished"
}

# --- Entry Point ---
main "$@"
```


[↑ Goto TOC](#table-of-contents)

### Strict Mode

Add this near the top of every script:

```bash
set -euo pipefail
```

- `-e`: Exit immediately if a command fails.
- `-u`: Treat unset variables as errors.
- `-o pipefail`: A pipeline fails if any command in it fails.


[↑ Goto TOC](#table-of-contents)

### Making a Script Executable

```bash
chmod +x my_script.sh
./my_script.sh          # Run it
```


[↑ Goto TOC](#table-of-contents)

### Quoting

Always quote variables to handle spaces and special characters:

```bash
# Bad — breaks if filename has spaces
rm $filename

# Good
rm "$filename"
```


[↑ Goto TOC](#table-of-contents)

### Error Handling

```bash
if ! cp "$src" "$dest"; then
    echo "Error: Failed to copy $src to $dest" >&2
    exit 1
fi
```

---


[↑ Goto TOC](#table-of-contents)

## 17. Useful Built-in Commands

```bash
echo "Hello"                # Print text
printf "%-10s %d\n" "hi" 5  # Formatted print
read -p "Input: " var        # Read user input
exit 0                       # Exit script with a status code
source ~/.bashrc             # Execute a file in the current shell
eval "echo hello"            # Evaluate a string as a command
type ls                      # Show what type of command something is
which python3                # Show the path of an executable
alias ll='ls -lah'           # Create a command shortcut
unalias ll                   # Remove an alias
history                      # Show command history
!42                          # Re-run command number 42 from history
!!                           # Re-run the last command
sudo !!                      # Re-run the last command as root
```

---


[↑ Goto TOC](#table-of-contents)

## 18. Process Management

```bash
ps aux                  # List all running processes
top                     # Live view of processes (press q to quit)
htop                    # Better version of top (if installed)
kill 1234               # Kill process by PID
kill -9 1234            # Force kill
killall firefox         # Kill all processes named firefox
pkill -f "my_script"    # Kill by name pattern

# Background and foreground
sleep 30 &              # Run in background; prints the PID
jobs                    # List background jobs
fg %1                   # Bring job 1 to foreground
bg %1                   # Resume job 1 in background
Ctrl+Z                  # Suspend a running process
Ctrl+C                  # Interrupt/kill a running process

# Keep running after logout
nohup ./script.sh &
```

---


[↑ Goto TOC](#table-of-contents)

## 19. Environment Variables

```bash
env                             # List all environment variables
echo $PATH                      # See your PATH
echo $HOME                      # Your home directory
echo $USER                      # Current username
echo $SHELL                     # Current shell

# Set a temporary variable (only for this session)
export MY_VAR="hello"

# Make it permanent — add to ~/.bashrc or ~/.bash_profile
echo 'export MY_VAR="hello"' >> ~/.bashrc
source ~/.bashrc

# Unset a variable
unset MY_VAR

# Modify PATH to add a custom directory
export PATH="$PATH:/home/user/my_scripts"
```

---


[↑ Goto TOC](#table-of-contents)

## 20. Tips and Tricks


[↑ Goto TOC](#table-of-contents)

### Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| `Ctrl+C` | Interrupt/kill current command |
| `Ctrl+Z` | Suspend current command |
| `Ctrl+D` | Send EOF (exit shell) |
| `Ctrl+L` | Clear the screen (`clear`) |
| `Ctrl+A` | Move cursor to beginning of line |
| `Ctrl+E` | Move cursor to end of line |
| `Ctrl+U` | Delete everything before cursor |
| `Ctrl+K` | Delete everything after cursor |
| `Ctrl+R` | Reverse search through history |
| `Tab` | Auto-complete commands and filenames |
| `↑` / `↓` | Cycle through command history |


[↑ Goto TOC](#table-of-contents)

### Brace Expansion

```bash
echo {a,b,c}.txt            # a.txt b.txt c.txt
echo file_{1..5}.txt        # file_1.txt file_2.txt ... file_5.txt
mkdir -p project/{src,tests,docs}   # Create multiple dirs at once
cp file.txt{,.bak}          # Quickly create a backup copy
```


[↑ Goto TOC](#table-of-contents)

### Here Documents

Embed multi-line text directly in a script:

```bash
cat <<EOF
This is line one
This is line two
Hello, $USER!
EOF

# Suppress variable expansion with quoted delimiter
cat <<'EOF'
This $USER will NOT be expanded.
EOF
```


[↑ Goto TOC](#table-of-contents)

### The `xargs` Command

Convert lines of input into arguments for another command:

```bash
find . -name "*.log" | xargs rm          # Delete all .log files found
cat urls.txt | xargs -I {} curl -O {}    # Download each URL from a file
```


[↑ Goto TOC](#table-of-contents)

### Useful One-Liners

```bash
# Count occurrences of a word in a file
grep -o "word" file.txt | wc -l

# Find and replace text in multiple files
sed -i 's/old/new/g' *.txt

# Print a specific column from a CSV
awk -F',' '{print $2}' data.csv

# Show disk usage of directories, sorted
du -sh */ | sort -h

# Generate a random password
openssl rand -base64 16

# Check if a website is up
curl -Is https://example.com | head -1

# Repeat a command every 2 seconds
watch -n 2 "free -h"
```

---


[↑ Goto TOC](#table-of-contents)

## Next Steps

You now have a solid foundation in Bash! Here are some suggestions for where to go next:

- **Practice** — The best way to learn is to use the terminal daily. Try completing real tasks in Bash instead of using a GUI.
- **Read man pages** — Type `man <command>` to see the full documentation for any command.
- **Explore `sed` and `awk`** — Powerful text-processing tools that pair perfectly with Bash.
- **Learn `cron`** — Schedule your scripts to run automatically with `crontab -e`.
- **Study real scripts** — Read scripts in `/etc/` or on GitHub to see how others solve problems.
- **ShellCheck** — Use [shellcheck.net](https://www.shellcheck.net) to check your scripts for bugs and bad practices.

Happy scripting! 🚀

[↑ Goto TOC](#table-of-contents)

---

© 2026 Jaco Steyn — Licensed under [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/)
