# Regular Expressions: A Complete Beginner's Guide

Regular expressions (regex or regexp) are patterns used to search, match, and manipulate text. Once you understand them, they're one of the most powerful tools in any programmer's toolkit. This guide will take you from zero to functional in a logical, step-by-step way.

---

## Table of Contents

1. [What Is a Regular Expression?](#what-is-a-regular-expression)
2. [How to Read This Guide](#how-to-read-this-guide)
3. [Literal Characters — The Basics](#literal-characters)
4. [Special Characters (Metacharacters)](#metacharacters)
5. [Character Classes](#character-classes)
6. [Shorthand Character Classes](#shorthand-character-classes)
7. [Anchors](#anchors)
8. [Quantifiers](#quantifiers)
9. [Grouping and Capturing](#grouping-and-capturing)
10. [Alternation](#alternation)
11. [Lookahead and Lookbehind](#lookahead-and-lookbehind)
12. [Flags / Modifiers](#flags)
13. [Common Patterns You Can Use Today](#common-patterns)
14. [Putting It All Together](#putting-it-all-together)
15. [Quick Reference Cheat Sheet](#cheat-sheet)

---

## 1. What Is a Regular Expression? {#what-is-a-regular-expression}

A regular expression is a **sequence of characters that defines a search pattern**. You can use it to:

- **Find** whether text matches a pattern (e.g., "does this string look like an email?")
- **Extract** parts of text (e.g., pull all phone numbers from a document)
- **Replace** text (e.g., swap all American date formats for European ones)
- **Validate** input (e.g., check that a password meets complexity rules)

Regex is supported in almost every programming language (Python, JavaScript, Java, Ruby, Go, etc.) and in many text editors (VS Code, Sublime Text, Notepad++).

### A Simple Example

Imagine you have the sentence:

```
The cat sat on the mat.
```

The regex `cat` matches the word "cat" in that string. Simple enough. But regex really shines when you need patterns, not just fixed words. The regex `[cm]at` matches both "cat" and "mat" — because `[cm]` means "either c or m."

---

## 2. How to Read This Guide {#how-to-read-this-guide}

Throughout this guide, patterns look like this: `/pattern/`

The slashes are a common way to delimit regex patterns (used in JavaScript, Perl, etc.). In Python you'd write `r"pattern"` instead. The slashes themselves are **not** part of the pattern.

When showing what a regex matches, matched text will be shown in **[brackets]** like:

```
Input:  The cat sat on the mat.
Match:  The [cat] sat on the [mat].
```

You can test all examples interactively at **regex101.com** — paste in a pattern and some text and it shows you exactly what matches and why.

---

## 3. Literal Characters {#literal-characters}

The simplest regex is just a sequence of literal characters. The pattern matches exactly those characters, in that order.

| Pattern | Matches in "I love regex" |
|---------|--------------------------|
| `/love/` | "I **love** regex" |
| `/regex/` | "I love **regex**" |
| `/ove r/` | "I l**ove r**egex" |

**Key point:** By default, regex is **case-sensitive**. `/Love/` would *not* match "love". (We'll cover how to change this with flags later.)

---

## 4. Special Characters (Metacharacters) {#metacharacters}

Some characters have special meaning in regex. These are called **metacharacters**:

```
. * + ? ^ $ { } [ ] | ( ) \
```

If you want to match one of these characters *literally*, you must **escape** it with a backslash `\`.

| You want to match | Use this pattern |
|-------------------|-----------------|
| A literal dot `.` | `\.` |
| A literal asterisk `*` | `\*` |
| A literal parenthesis `(` | `\(` |
| A literal backslash `\` | `\\` |

### The Dot `.`

The dot `.` is the wildcard character. It matches **any single character** except a newline.

```
Pattern:  /c.t/
Input:    cat, cot, cut, c3t, c-t
Matches:  [cat], [cot], [cut], [c3t], [c-t]
```

The dot does **not** match "ct" (no character between c and t) or "coat" (two characters between c and t).

---

## 5. Character Classes {#character-classes}

A **character class** is a set of characters inside square brackets `[ ]`. It matches **one character** from the set.

### Basic Character Classes

| Pattern | Matches |
|---------|---------|
| `[aeiou]` | Any single vowel |
| `[abc]` | Either a, b, or c |
| `[0123456789]` | Any single digit |

```
Pattern:  /[cm]at/
Input:    The cat sat on the mat.
Matches:  The [cat] sat on the [mat].
```

### Ranges

Inside a character class, you can use a **hyphen `-`** to specify a range of characters:

| Pattern | Matches |
|---------|---------|
| `[a-z]` | Any lowercase letter |
| `[A-Z]` | Any uppercase letter |
| `[0-9]` | Any digit |
| `[a-zA-Z]` | Any letter, upper or lower |
| `[a-zA-Z0-9]` | Any letter or digit |

### Negated Character Classes

Put a caret `^` at the **start** of a character class to match anything *except* those characters:

| Pattern | Matches |
|---------|---------|
| `[^aeiou]` | Any character that is NOT a vowel |
| `[^0-9]` | Any character that is NOT a digit |

```
Pattern:  /[^aeiou]/
Input:    hello
Matches:  [h]e[ll][o]  →  h, l, l are matched
```

> ⚠️ The caret `^` has different meanings in different places:
> - Inside `[ ]` at the start: means "not"
> - Outside `[ ]`: means "start of line" (covered in Anchors)

---

## 6. Shorthand Character Classes {#shorthand-character-classes}

Because some character classes are used so often, regex provides shorthand codes for them:

| Shorthand | Equivalent | Matches |
|-----------|-----------|---------|
| `\d` | `[0-9]` | Any digit |
| `\D` | `[^0-9]` | Any non-digit |
| `\w` | `[a-zA-Z0-9_]` | Any "word" character (letters, digits, underscore) |
| `\W` | `[^a-zA-Z0-9_]` | Any non-word character |
| `\s` | `[ \t\n\r\f]` | Any whitespace (space, tab, newline, etc.) |
| `\S` | `[^ \t\n\r\f]` | Any non-whitespace character |

```
Pattern:  /\d\d\d/
Input:    My ZIP code is 90210.
Matches:  My ZIP code is [902]10.
          (matches first 3 digits)
```

---

## 7. Anchors {#anchors}

Anchors don't match characters — they match **positions** in the string.

| Anchor | Matches the position... |
|--------|------------------------|
| `^` | At the **start** of the string (or line) |
| `$` | At the **end** of the string (or line) |
| `\b` | At a **word boundary** (transition between `\w` and `\W`) |
| `\B` | At a **non-word boundary** |

### Start and End Anchors

```
Pattern:  /^Hello/
Input:    Hello, world!
Match:    [Hello], world!    ✅ matches (Hello is at start)

Pattern:  /^Hello/
Input:    Say Hello!
Match:    No match           ❌ Hello is not at start
```

```
Pattern:  /world!$/
Input:    Hello, world!
Match:    Hello, [world!]    ✅ matches (world! is at end)
```

### Word Boundaries

`\b` is incredibly useful for matching whole words rather than partial matches:

```
Pattern:  /cat/
Input:    The cat scattered across the catfish.
Matches:  The [cat] s[cat]tered across the [cat]fish.
          (matches "cat" everywhere, including inside other words)

Pattern:  /\bcat\b/
Input:    The cat scattered across the catfish.
Matches:  The [cat] scattered across the catfish.
          (only matches "cat" as a complete word)
```

---

## 8. Quantifiers {#quantifiers}

Quantifiers specify **how many times** the preceding element must appear.

| Quantifier | Meaning |
|-----------|---------|
| `*` | Zero or more |
| `+` | One or more |
| `?` | Zero or one (makes it optional) |
| `{n}` | Exactly n times |
| `{n,}` | At least n times |
| `{n,m}` | Between n and m times (inclusive) |

### Examples

```
Pattern:  /go*/
Matches:  "g", "go", "goo", "gooo", "goooo", ...
(zero or more "o"s after "g")

Pattern:  /go+/
Matches:  "go", "goo", "gooo", ...
(one or more "o"s — so "g" alone does NOT match)

Pattern:  /colou?r/
Matches:  "color" and "colour"
(the "u" is optional)

Pattern:  /\d{3}-\d{4}/
Matches:  "555-1234"
(exactly 3 digits, a dash, exactly 4 digits)

Pattern:  /\d{2,4}/
Matches:  "12", "123", "1234"
(between 2 and 4 digits)
```

### Greedy vs. Lazy

By default, quantifiers are **greedy** — they match as much as possible.

```
Pattern:  /<.+>/
Input:    <div>Hello</div>
Match:    [<div>Hello</div>]
          (greedy: matches from first < to LAST >)
```

Add `?` after a quantifier to make it **lazy** — match as little as possible:

```
Pattern:  /<.+?>/
Input:    <div>Hello</div>
Match:    [<div>]Hello[</div>]
          (lazy: matches each tag separately)
```

| Greedy | Lazy |
|--------|------|
| `*` | `*?` |
| `+` | `+?` |
| `?` | `??` |
| `{n,m}` | `{n,m}?` |

---

## 9. Grouping and Capturing {#grouping-and-capturing}

Parentheses `( )` serve two purposes: **grouping** and **capturing**.

### Grouping

Group part of a pattern so you can apply a quantifier to the whole group:

```
Pattern:  /(ab)+/
Matches:  "ab", "abab", "ababab"
(the + applies to the whole group "ab", not just "b")

Pattern:  /gr(a|e)y/
Matches:  "gray" and "grey"
```

### Capturing

When you use `( )`, the regex engine **captures** (remembers) what that group matched. You can then reference it:

- In a **replacement string**, use `$1`, `$2`, etc. (JavaScript) or `\1`, `\2`, etc. (Python)
- In the **pattern itself**, use a **backreference** like `\1`

**Example — Rearranging a date:**

```
Pattern:  /(\d{4})-(\d{2})-(\d{2})/
Input:    2024-03-15
Groups:   Group 1 = "2024", Group 2 = "03", Group 3 = "15"
Replace:  $3/$2/$1  →  "15/03/2024"
```

**Example — Backreference (find doubled words):**

```
Pattern:  /\b(\w+)\s+\1\b/
Input:    This is is a test test.
Matches:  This [is is] a [test test].
(\1 matches whatever group 1 captured)
```

### Non-Capturing Groups

If you just want to group without capturing, use `(?:...)`:

```
Pattern:  /(?:ab)+/
          (groups "ab" for the quantifier, but doesn't capture it)
```

This is slightly more efficient and keeps your group numbering clean.

---

## 10. Alternation {#alternation}

The pipe `|` means "or." It matches either the expression to its left or the expression to its right.

```
Pattern:  /cat|dog/
Matches:  "cat" or "dog"

Pattern:  /I have a cat|dog/
Matches:  "I have a cat" or "dog"
          ⚠️ Alternation has very low precedence!

Pattern:  /I have a (cat|dog)/
Matches:  "I have a cat" or "I have a dog"
          ✅ Use groups to control scope
```

---

## 11. Lookahead and Lookbehind {#lookahead-and-lookbehind}

**Lookarounds** let you match something only if it is (or isn't) preceded or followed by something else — without including that something else in the match. They are **zero-width** assertions (they don't consume characters).

### Positive Lookahead `(?=...)`

Match X only if followed by Y:

```
Pattern:  /\d+(?= dollars)/
Input:    I have 50 dollars and 30 euros.
Match:    I have [50] dollars and 30 euros.
(matches "50" only because it's followed by " dollars")
```

### Negative Lookahead `(?!...)`

Match X only if NOT followed by Y:

```
Pattern:  /\d+(?! dollars)/
Input:    I have 50 dollars and 30 euros.
Match:    I have 50 dollars and [30] euros.
(matches "30" because it is NOT followed by " dollars")
```

### Positive Lookbehind `(?<=...)`

Match X only if preceded by Y:

```
Pattern:  /(?<=\$)\d+/
Input:    The price is $42.
Match:    The price is $[42].
(matches "42" only because it's preceded by "$")
```

### Negative Lookbehind `(?<!...)`

Match X only if NOT preceded by Y:

```
Pattern:  /(?<!\$)\d+/
Input:    I have $100 and 50 apples.
Match:    I have $100 and [50] apples.
(matches "50" because it's NOT preceded by "$")
```

> ⚠️ Lookbehind support varies. Most modern engines support it, but older JavaScript (pre-2018) does not.

---

## 12. Flags / Modifiers {#flags}

Flags change how the entire pattern behaves. They're placed after the closing `/` in languages that use slash delimiters.

| Flag | Name | Effect |
|------|------|--------|
| `i` | Case insensitive | `/hello/i` matches "hello", "Hello", "HELLO" |
| `g` | Global | Find all matches, not just the first |
| `m` | Multiline | `^` and `$` match start/end of each **line**, not just the whole string |
| `s` | Dotall | Makes `.` match **newlines** too |
| `x` | Extended / Verbose | Allows whitespace and comments in pattern (for readability) |

### Examples

```javascript
// JavaScript examples
"Hello WORLD".match(/hello/i)   // matches "Hello" (case insensitive)
"cat cat cat".match(/cat/g)     // matches all three "cat"s (global)
```

```python
# Python examples
import re
re.findall(r"cat", "CAT cat Cat", re.IGNORECASE)  # ['CAT', 'cat', 'Cat']
```

---

## 13. Common Patterns You Can Use Today {#common-patterns}

Here are battle-tested regex patterns for everyday use. Don't worry about understanding every detail — you can learn from them over time.

### Email Address (simple)
```
/^[a-zA-Z0-9._%+\-]+@[a-zA-Z0-9.\-]+\.[a-zA-Z]{2,}$/
```

### URL
```
/https?:\/\/[^\s/$.?#].[^\s]*/
```

### Phone Number (US format)
```
/\(?\d{3}\)?[\s.\-]?\d{3}[\s.\-]?\d{4}/
```
Matches: (555) 123-4567, 555.123.4567, 5551234567

### Date (YYYY-MM-DD)
```
/^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$/
```

### IPv4 Address
```
/^(\d{1,3}\.){3}\d{1,3}$/
```

### Hex Color Code
```
/#([a-fA-F0-9]{6}|[a-fA-F0-9]{3})/
```
Matches: #FF5733, #fff

### ZIP Code (US)
```
/^\d{5}(-\d{4})?$/
```

### Strong Password (8+ chars, upper, lower, digit, special char)
```
/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/
```

### Whitespace-only lines
```
/^\s*$/
```

### Trim leading/trailing whitespace
```
Pattern to find: /^\s+|\s+$/g
Replace with:    (empty string)
```

---

## 14. Putting It All Together {#putting-it-all-together}

Let's walk through building a regex from scratch for a real problem: **extracting all hashtags from a social media post.**

**Input:**
```
Loving this #sunset view! #travel #photography2024 always inspires me.
```

**What we need to match:**
- A `#` character
- Followed by one or more word characters (letters, digits, underscores)
- But only when `#` appears at a word boundary (not inside another word)

**Step 1** — Match the `#`:
```
/#/
```

**Step 2** — Match one or more word characters after it:
```
/#\w+/
```

**Step 3** — Make sure `#` is at a word boundary (so we don't match things like `c#` inside code):
```
/\B#\w+/
```
(`\B` is a non-word boundary, which is what exists before `#` when it starts a hashtag)

**Result:**
```
Pattern:  /\B#\w+/g
Input:    Loving this #sunset view! #travel #photography2024 always inspires me.
Matches:  [#sunset], [#travel], [#photography2024]
```

---

## 15. Quick Reference Cheat Sheet {#cheat-sheet}

### Characters

| Pattern | Matches |
|---------|---------|
| `abc` | Literal "abc" |
| `.` | Any character except newline |
| `\.` | A literal dot |
| `\n` | Newline |
| `\t` | Tab |

### Character Classes

| Pattern | Matches |
|---------|---------|
| `[abc]` | a, b, or c |
| `[^abc]` | Anything except a, b, c |
| `[a-z]` | Any lowercase letter |
| `[A-Z]` | Any uppercase letter |
| `[0-9]` | Any digit |
| `\d` | Any digit (= `[0-9]`) |
| `\D` | Any non-digit |
| `\w` | Word character `[a-zA-Z0-9_]` |
| `\W` | Non-word character |
| `\s` | Whitespace |
| `\S` | Non-whitespace |

### Anchors

| Pattern | Position |
|---------|---------|
| `^` | Start of string/line |
| `$` | End of string/line |
| `\b` | Word boundary |
| `\B` | Non-word boundary |

### Quantifiers

| Pattern | Quantity |
|---------|---------|
| `*` | 0 or more |
| `+` | 1 or more |
| `?` | 0 or 1 |
| `{n}` | Exactly n |
| `{n,}` | n or more |
| `{n,m}` | Between n and m |
| `*?` `+?` | Lazy versions |

### Groups

| Pattern | Meaning |
|---------|---------|
| `(abc)` | Capturing group |
| `(?:abc)` | Non-capturing group |
| `(a\|b)` | Alternation (a or b) |
| `(?=abc)` | Positive lookahead |
| `(?!abc)` | Negative lookahead |
| `(?<=abc)` | Positive lookbehind |
| `(?<!abc)` | Negative lookbehind |

### Common Flags

| Flag | Meaning |
|------|---------|
| `i` | Case insensitive |
| `g` | Global (find all matches) |
| `m` | Multiline |
| `s` | Dotall (`.` matches newline) |

---

## Next Steps

Now that you have the fundamentals, here's how to keep improving:

1. **Practice on regex101.com** — the best free tool for learning and debugging regex. It explains your pattern step by step.
2. **Use regexr.com** for community examples and a clean interface.
3. **Read error messages carefully** — most regex engines tell you exactly where a pattern fails.
4. **Build patterns incrementally** — start simple and add complexity piece by piece.
5. **Don't memorize everything** — keep this cheat sheet handy and look things up. Even experienced developers do.

The biggest leap in regex mastery comes from reading other people's patterns and understanding *why* they're written the way they are. Good luck!
