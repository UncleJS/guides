# A Beginner's Guide to `jq`

`jq` is a lightweight, powerful command-line tool for parsing, filtering, and transforming JSON data. Think of it as `sed` or `awk` for JSON — it lets you slice, dice, and reshape JSON right in your terminal.

---

## Table of Contents

1. [Installation](#installation)
2. [Basic Usage](#basic-usage)
3. [Understanding Filters](#understanding-filters)
4. [Accessing Data](#accessing-data)
5. [Working with Arrays](#working-with-arrays)
6. [Built-in Functions](#built-in-functions)
7. [Transforming Data](#transforming-data)
8. [Conditionals and Logic](#conditionals-and-logic)
9. [Advanced Features](#advanced-features)
10. [Real-World Examples](#real-world-examples)
11. [Quick Reference Cheat Sheet](#quick-reference-cheat-sheet)

---

## Installation

**macOS**
```bash
brew install jq
```

**Ubuntu / Debian**
```bash
sudo apt-get install jq
```

**Fedora / RHEL**
```bash
sudo dnf install jq
```

**Windows (via Chocolatey)**
```bash
choco install jq
```

**Windows (via Winget)**
```bash
winget install jqlang.jq
```

Verify your installation:
```bash
jq --version
# jq-1.7.1
```

---

## Basic Usage

The basic syntax of `jq` is:

```bash
jq '<filter>' <file>
# or pipe JSON into it:
echo '<json>' | jq '<filter>'
```

The simplest filter is `.` (dot), which just pretty-prints the input:

```bash
echo '{"name":"Alice","age":30}' | jq '.'
```

Output:
```json
{
  "name": "Alice",
  "age": 30
}
```

### Useful Flags

| Flag | Description |
|------|-------------|
| `-r` | Raw output (strips quotes from strings) |
| `-c` | Compact output (single line) |
| `-n` | Null input (don't read input, useful with `now` or constructing data) |
| `-e` | Exit with a non-zero status if the last output is `false` or `null` |
| `-s` | Slurp all input lines into a single array |
| `--arg name val` | Pass a shell variable into jq as a string |
| `--argjson name val` | Pass a shell variable into jq as a JSON value |
| `-f file` | Read filter from a file |

---

## Understanding Filters

Everything in `jq` is a **filter** — an expression that takes an input and produces an output. Filters can be chained together with the pipe operator `|`, just like shell pipes.

```bash
# Chain filters with |
echo '{"user":{"name":"Alice","age":30}}' | jq '.user | .name'
# "Alice"
```

Multiple outputs can be produced from a single input using `,`:

```bash
echo '{"name":"Alice","age":30}' | jq '.name, .age'
# "Alice"
# 30
```

---

## Accessing Data

### Object Fields

Use `.fieldname` to access a field:

```bash
echo '{"name":"Alice","age":30}' | jq '.name'
# "Alice"
```

Use `.["fieldname"]` for fields with special characters or spaces:

```bash
echo '{"first-name":"Alice"}' | jq '.["first-name"]'
# "Alice"
```

### Nested Fields

Chain dots to go deeper:

```bash
echo '{"user":{"address":{"city":"London"}}}' | jq '.user.address.city'
# "London"
```

### Optional Operator

Append `?` to suppress errors if a field doesn't exist:

```bash
echo '{"name":"Alice"}' | jq '.age?'
# (no output, no error)
```

### Recursive Descent (`..`)

`..` recursively descends into all values. Useful for searching nested structures:

```bash
echo '{"a":{"b":{"c":42}}}' | jq '.. | numbers'
# 42
```

---

## Working with Arrays

### Sample Data

Let's use this JSON for array examples:

```json
[
  {"name": "Alice", "age": 30, "city": "London"},
  {"name": "Bob",   "age": 25, "city": "Paris"},
  {"name": "Carol", "age": 35, "city": "London"}
]
```

Save it as `people.json`.

### Array Indexing

```bash
# First element (0-indexed)
jq '.[0]' people.json

# Last element
jq '.[-1]' people.json

# Slice (elements 0 and 1)
jq '.[0:2]' people.json
```

### Iterating with `.[]`

`.[]` explodes an array into individual elements:

```bash
jq '.[]' people.json
# outputs each object one by one
```

Combine with a field access to extract a specific field from every element:

```bash
jq '.[].name' people.json
# "Alice"
# "Bob"
# "Carol"
```

### `map()`

`map(f)` applies a filter `f` to every element and returns an array:

```bash
jq 'map(.name)' people.json
# ["Alice", "Bob", "Carol"]
```

### `select()`

`select(condition)` filters elements, keeping only those where the condition is true:

```bash
jq '[.[] | select(.age > 28)]' people.json
# [{"name":"Alice",...}, {"name":"Carol",...}]
```

This is equivalent to:
```bash
jq 'map(select(.age > 28))' people.json
```

### `length`

```bash
jq 'length' people.json         # 3 (number of elements)
jq '.[0].name | length' people.json  # 5 (string length)
```

### `first` and `last`

```bash
jq 'first' people.json
jq 'last' people.json
```

---

## Built-in Functions

### String Functions

```bash
echo '"  hello world  "' | jq 'ltrimstr(" ") | rtrimstr(" ")'
echo '"hello"' | jq 'ascii_upcase'          # "HELLO"
echo '"HELLO"' | jq 'ascii_downcase'         # "hello"
echo '"hello world"' | jq 'split(" ")'       # ["hello","world"]
echo '["hello","world"]' | jq 'join(", ")'   # "hello, world"
echo '"hello"' | jq 'startswith("hel")'      # true
echo '"hello"' | jq 'endswith("lo")'         # true
echo '"hello world"' | jq 'test("world")'    # true (regex match)
```

### Number Functions

```bash
echo '3.7' | jq 'floor'    # 3
echo '3.2' | jq 'ceil'     # 4
echo '3.7' | jq 'round'    # 4
echo '-5'  | jq 'fabs'     # 5 (absolute value)
echo '2'   | jq 'sqrt'     # 1.4142...
```

### Array Functions

```bash
echo '[3,1,2]' | jq 'sort'              # [1,2,3]
jq 'sort_by(.age)' people.json          # sort objects by field
jq 'sort_by(.age) | reverse' people.json  # descending
echo '[1,2,2,3]' | jq 'unique'          # [1,2,3]
jq 'unique_by(.city)' people.json       # unique by field
echo '[[1,2],[3,4]]' | jq 'flatten'     # [1,2,3,4]
echo '[1,2,3]' | jq 'reverse'           # [3,2,1]
echo '[1,2,3]' | jq 'min'              # 1
echo '[1,2,3]' | jq 'max'              # 3
jq 'min_by(.age)' people.json          # object with min age
jq 'group_by(.city)' people.json       # group into arrays by field
```

### Object Functions

```bash
echo '{"a":1,"b":2}' | jq 'keys'       # ["a","b"]
echo '{"a":1,"b":2}' | jq 'values'     # [1,2]
echo '{"a":1,"b":2}' | jq 'has("a")'   # true
echo '{"a":1,"b":2}' | jq 'to_entries' 
# [{"key":"a","value":1},{"key":"b","value":2}]
echo '[{"key":"a","value":1}]' | jq 'from_entries'
# {"a":1}
echo '{"a":1,"b":2}' | jq 'with_entries(.value += 10)'
# {"a":11,"b":12}
```

### Type Functions

```bash
echo '42' | jq 'type'          # "number"
echo '"hi"' | jq 'type'        # "string"
echo 'null' | jq 'type'        # "null"
echo '[]' | jq 'type'          # "array"
echo '{}' | jq 'type'          # "object"

# Type conversion
echo '"42"' | jq 'tonumber'    # 42
echo '42' | jq 'tostring'      # "42"
```

---

## Transforming Data

### Building New Objects

Use `{key: value}` to construct new objects:

```bash
jq '.[] | {person: .name, location: .city}' people.json
# {"person":"Alice","location":"London"}
# {"person":"Bob","location":"Paris"}
# ...
```

Wrap in `[]` to collect into an array:

```bash
jq '[.[] | {person: .name, location: .city}]' people.json
```

### Shorthand for Same-Name Keys

If the key and field name are the same, you can use shorthand:

```bash
# These are equivalent:
jq '{name: .name, age: .age}' people.json
jq '{name, age}' people.json
```

### Adding / Updating Fields

Use `+` to merge objects:

```bash
jq '.[] | . + {country: "Europe"}' people.json
```

Or use the update operator `|=`:

```bash
# Add a field to each element
jq '[.[] | . + {active: true}]' people.json

# Modify an existing field
jq '[.[] | .age |= . + 1]' people.json
```

### Deleting Fields

```bash
jq 'map(del(.city))' people.json
```

### String Interpolation

Use `\(.expression)` inside strings:

```bash
jq '.[] | "\(.name) lives in \(.city)"' people.json
# "Alice lives in London"
# "Bob lives in Paris"
# "Carol lives in London"
```

---

## Conditionals and Logic

### `if-then-else`

```bash
jq '.[] | if .age >= 30 then "\(.name) is senior" else "\(.name) is junior" end' people.json
```

### Comparison Operators

```
==  !=  <  <=  >  >=
```

### Boolean Operators

```
and  or  not
```

```bash
jq '.[] | select(.age > 25 and .city == "London")' people.json
```

### Alternative Operator `//`

The `//` operator returns the right side if the left side is `false` or `null`:

```bash
echo '{"name": null}' | jq '.name // "Unknown"'
# "Unknown"

echo '{}' | jq '.missing // "default"'
# "default"
```

---

## Advanced Features

### Reduce

`reduce` is like a fold/accumulator — it processes each element and accumulates a result:

```bash
# Sum all ages
jq 'reduce .[].age as $age (0; . + $age)' people.json
# 90

# Build a lookup map: name -> age
jq 'reduce .[] as $p ({}; . + {($p.name): $p.age})' people.json
# {"Alice":30,"Bob":25,"Carol":35}
```

### Variables

Use `as $varname` to store intermediate values:

```bash
jq '.[] | .name as $n | .age as $a | "\($n) is \($a)"' people.json
```

### `any` and `all`

```bash
jq 'any(.[]; .age > 30)' people.json   # true (Carol is 35)
jq 'all(.[]; .age > 20)' people.json   # true
```

### `env` — Reading Environment Variables

```bash
export CITY=London
jq --arg city "$CITY" '[.[] | select(.city == $city)]' people.json
```

Or directly with `$ENV`:

```bash
jq '[.[] | select(.city == $ENV.CITY)]' people.json
```

### `@base64` and `@uri`

```bash
echo '"hello world"' | jq '@uri'      # "hello%20world"
echo '"hello"' | jq '@base64'         # "aGVsbG8="
echo '"aGVsbG8="' | jq '@base64d'     # "hello"
```

### `@csv` and `@tsv`

```bash
jq -r '.[] | [.name, .age, .city] | @csv' people.json
# "Alice",30,"London"
# "Bob",25,"Paris"
# "Carol",35,"London"

jq -r '.[] | [.name, .age, .city] | @tsv' people.json
```

### `@json` — Embed JSON as a String

```bash
echo '{"a":1}' | jq '@json'
# "{\"a\":1}"
```

### `limit`

```bash
# Get first 2 results
jq '[limit(2; .[])]' people.json
```

### `path()`

Returns the path to a value as an array:

```bash
echo '{"a":{"b":1}}' | jq 'path(.a.b)'
# ["a","b"]
```

### `getpath` / `setpath` / `delpaths`

```bash
echo '{"a":{"b":1}}' | jq 'getpath(["a","b"])'  # 1
echo '{"a":{"b":1}}' | jq 'setpath(["a","b"]; 99)'  # {"a":{"b":99}}
```

---

## Real-World Examples

### Parse API Responses

```bash
# Get all GitHub repo names from user
curl -s https://api.github.com/users/octocat/repos | jq '.[].name'

# Get name + stargazer count
curl -s https://api.github.com/users/octocat/repos \
  | jq '[.[] | {name, stars: .stargazers_count}] | sort_by(.stars) | reverse'
```

### Process Log Files (JSON Logs)

```bash
# Filter only error logs
cat app.log | jq 'select(.level == "error")'

# Count errors by service
cat app.log | jq -s 'group_by(.service) | map({service: .[0].service, count: length})'
```

### Transform CSV-like Data

```bash
# Convert array of objects to CSV
jq -r '["name","age","city"], (.[] | [.name, .age, .city]) | @csv' people.json
```

### Merge Two JSON Files

```bash
jq -s '.[0] * .[1]' file1.json file2.json
```

### Extract and Format Config Values

```bash
# Get all enabled features
jq '[.features[] | select(.enabled == true) | .name]' config.json
```

### Count Occurrences

```bash
# Count people per city
jq 'group_by(.city) | map({city: .[0].city, count: length})' people.json
```

### Flatten Nested API Data

```bash
# Extract nested list items
jq '[.results[].items[] | {id, title}]' response.json
```

---

## Quick Reference Cheat Sheet

```
BASICS
.                   Identity / pretty-print
.foo                Object field access
.foo.bar            Nested field access
.["foo"]            Field with special chars
.[0]                Array index
.[-1]               Last array element
.[2:5]              Array slice

ITERATION & FILTERING
.[]                 Iterate array or object values
map(f)              Apply filter to each element
select(cond)        Keep elements matching condition
empty               Produce no output

CONSTRUCTING
{a, b}              Build object (shorthand)
{a: .x, b: .y}     Build object (explicit)
[expr]              Collect into array

OPERATORS
|                   Pipe (chain filters)
,                   Multiple outputs
//                  Alternative (default if null/false)
+                   Add / merge
-                   Subtract / difference
*                   Multiply / string repeat
/                   Divide
%                   Modulo
|=                  Update in place

STRINGS
"\(.foo)"           String interpolation
ltrimstr(s)         Remove prefix
rtrimstr(s)         Remove suffix
split(s)            Split string
join(s)             Join array into string
test(regex)         Regex match (bool)
ascii_upcase        Uppercase
ascii_downcase      Lowercase

ARRAYS
length              Array/string length
sort                Sort array
sort_by(f)          Sort by field
unique              Remove duplicates
unique_by(f)        Unique by field
group_by(f)         Group into arrays
flatten             Flatten nested arrays
reverse             Reverse array
first, last         First/last element
min, max            Min/max value
min_by(f), max_by(f)  Min/max by field
any(gen; cond)      True if any match
all(gen; cond)      True if all match
add                 Sum/merge all elements
indices(s)          Find positions

OBJECTS
keys                Array of keys
values              Array of values
has("key")          Check key exists
in(obj)             Check if key in object
to_entries          [{key,value}] pairs
from_entries        {key:value} from pairs
with_entries(f)     Map over entries
del(.foo)           Delete field

TYPES
type                Get type string
tonumber            Convert to number
tostring            Convert to string
arrays              Select arrays
objects             Select objects
strings             Select strings
numbers             Select numbers
booleans            Select booleans
nulls               Select nulls
iterables           Select arrays/objects
scalars             Select non-iterables

ADVANCED
reduce expr as $x (init; update)   Fold
limit(n; expr)      First n results
recurse(f)          Recursive descent
env                 Environment object
$ENV.VAR            Env variable
@base64 / @base64d  Encode/decode base64
@uri                URL encode
@csv / @tsv         CSV/TSV format
@json               JSON string embed
path(expr)          Get path as array
getpath(path)       Get value at path
setpath(path; val)  Set value at path
input               Read next input
inputs              All remaining inputs
```

---

## Tips for Learning jq

**Start simple.** Don't try to write complex one-liners from the start. Build up your filter step by step using `|`.

**Use the jq playground.** Try [jqplay.org](https://jqplay.org) for an interactive browser-based environment.

**Wrap arrays manually.** If you want an array output, wrap your expression in `[...]`.

**Use `-r` for scripts.** When using `jq` output in shell scripts, `-r` removes JSON string quotes so you can use values directly.

**Validate your JSON first.** Run `jq '.' file.json` as a quick sanity check that your JSON is valid.

**Use `--arg` for shell variables.** Never interpolate shell variables directly into jq filters — use `--arg name value` instead to avoid injection issues.

---

*Happy filtering! Once you get comfortable with `jq`, you'll wonder how you ever worked with JSON without it.*
