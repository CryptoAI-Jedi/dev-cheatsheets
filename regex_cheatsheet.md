# Regex Cheatsheet

> Pattern matching across grep, sed, awk, Python, and jq. One language, several dialects. The syntax below is the common core, with flavor differences flagged where they bite.

---

## Table of Contents
- [Core Syntax](#core-syntax)
- [Quantifiers](#quantifiers)
- [Character Classes](#character-classes)
- [Anchors & Boundaries](#anchors--boundaries)
- [Groups & References](#groups--references)
- [Flags / Modifiers](#flags--modifiers)
- [Flavors (BRE / ERE / PCRE)](#flavors-bre--ere--pcre)
- [Python (re module)](#python-re-module)
- [grep Usage](#grep-usage)
- [sed Usage](#sed-usage)
- [awk Usage](#awk-usage)
- [Common Regex Patterns](#common-regex-patterns)
- [Testing Tools](#testing-tools)
- [Tips & Gotchas](#tips--gotchas)

---

## CORE SYNTAX

```text
.          → Any single character (except newline)
^          → Start of string / line
$          → End of string / line
\          → Escape special character  e.g. \. matches literal dot
|          → OR  e.g. cat|dog
()         → Capture group
(?:...)    → Non-capturing group
[]         → Character class  e.g. [aeiou]
[^]        → Negated character class  e.g. [^0-9]
```

---

## QUANTIFIERS

```text
*          → 0 or more
+          → 1 or more
?          → 0 or 1 (optional)
{n}        → Exactly n times
{n,}       → n or more times
{n,m}      → Between n and m times
```

### Greedy vs lazy

```text
.*         → Greedy (match as much as possible)
.*?        → Lazy (match as little as possible)
.+?        → Lazy one or more
??  {n,m}? → Lazy versions of ? and {n,m}
```

---

## CHARACTER CLASSES

```text
\d         → Digit [0-9]
\D         → Non-digit [^0-9]
\w         → Word character [a-zA-Z0-9_]
\W         → Non-word character
\s         → Whitespace (space, tab, newline)
\S         → Non-whitespace
\b         → Word boundary
\B         → Non-word boundary
\n         → Newline
\t         → Tab
\r         → Carriage return
```

### Custom classes

```text
[a-z]         → Lowercase letter
[A-Z]         → Uppercase letter
[0-9]         → Digit
[a-zA-Z]      → Any letter
[a-zA-Z0-9_]  → Word characters (same as \w)
[^aeiou]      → Not a vowel
```

---

## ANCHORS & BOUNDARIES

```text
^hello        → String starts with "hello"
world$        → String ends with "world"
^hello$       → Exact match "hello" only
\bword\b      → Whole word "word" (not "words" or "keyword")
\Bword\B      → "word" NOT at a word boundary
```

---

## GROUPS & REFERENCES

```text
(abc)         → Capture group 1
(abc)(def)    → Groups 1 and 2
\1            → Backreference to group 1
(?P<name>...) → Named group (Python)
(?<name>...)  → Named group (JS / PCRE / jq)
(?P=name)     → Backreference to named group (Python)
(?:abc)       → Non-capturing group (match but don't capture)
(?=abc)       → Positive lookahead (followed by abc)
(?!abc)       → Negative lookahead (NOT followed by abc)
(?<=abc)      → Positive lookbehind (preceded by abc)
(?<!abc)      → Negative lookbehind (NOT preceded by abc)
```

---

## FLAGS / MODIFIERS

```text
i     → Case-insensitive  /pattern/i
g     → Global (find all matches)  /pattern/g
m     → Multiline (^ and $ match line boundaries)
s     → Dotall (. matches newline too)
x     → Extended (allow whitespace/comments in pattern)
```

### In Python

```text
re.IGNORECASE  (re.I)
re.MULTILINE   (re.M)
re.DOTALL      (re.S)
re.VERBOSE     (re.X)
```

---

## FLAVORS (BRE / ERE / PCRE)

Same idea, different escaping — this is why a pattern "works in Python but not in sed":

```text
BRE  (grep, sed default)  → + ? { } ( ) | are LITERAL unless escaped: \+ \? \{3\} \( \) \|
ERE  (grep -E, sed -E)    → + ? {} () | work unescaped — like you'd expect
PCRE (grep -P, Python)    → Full modern syntax: \d, lookarounds, lazy, named groups
```

```text
💡 Rule of thumb: reach for grep -E / sed -E by default; use grep -P
when you need \d, lookarounds, or lazy quantifiers. \d does NOT work
in BRE/ERE — use [0-9].
```

---

## PYTHON (re module)

```python
import re

re.search(r'\d+', text)               # First match anywhere in string
re.match(r'\d+', text)                # Match at START of string only
re.fullmatch(r'\d+', text)            # Entire string must match
re.findall(r'\d+', text)              # List of all matches
re.finditer(r'\d+', text)             # Iterator of match objects
re.sub(r'\d+', 'NUM', text)           # Replace all matches
re.sub(r'\d+', 'NUM', text, count=1)  # Replace first only
re.split(r'\s+', text)                # Split on whitespace
re.compile(r'\d+')                    # Compile for reuse (faster in loops)
```

### Match object methods

```python
m = re.search(r'(\d+)', text)
m.group()             # Full match
m.group(1)            # Capture group 1
m.groups()            # All capture groups as tuple
m.start(); m.end()    # Match position
m.span()              # (start, end) tuple
```

### Named groups (Python)

```python
m = re.search(r'(?P<year>\d{4})-(?P<month>\d{2})', text)
m.group('year')
m.group('month')
```

---

## GREP USAGE

```text
grep "pattern" file                → Basic search
grep -i "pattern" file             → Case-insensitive
grep -r "pattern" /path            → Recursive
grep -n "pattern" file             → Show line numbers
grep -v "pattern" file             → Invert (lines NOT matching)
grep -c "pattern" file             → Count matching lines
grep -l "pattern" /path            → List files with matches
grep -o "pattern" file             → Print only matched part
grep -E "pattern" file             → Extended regex (ERE)
grep -P "pattern" file             → Perl-compatible regex (PCRE)
grep -A 3 "pattern" file           → 3 lines AFTER match
grep -B 3 "pattern" file           → 3 lines BEFORE match
grep -C 3 "pattern" file           → 3 lines before AND after
```

---

## SED USAGE

```text
sed 's/old/new/' file              → Replace first on each line
sed 's/old/new/g' file             → Replace all occurrences
sed 's/old/new/gi' file            → Case-insensitive replace all
sed -i 's/old/new/g' file          → In-place edit
sed -i.bak 's/old/new/g' file      → In-place with backup
sed -n '5,10p' file                → Print lines 5-10
sed '5,10d' file                   → Delete lines 5-10
sed '/pattern/d' file              → Delete lines matching pattern
sed -n '/pattern/p' file           → Print only matching lines
sed 's/\b[0-9]\{3\}\b/NUM/g' file  → Replace 3-digit numbers (BRE: \{3\})
sed -E 's/[0-9]{3}/NUM/g' file     → Same, ERE — no escaping ({3} plain)
```

---

## AWK USAGE

```text
awk '{print $1}' file              → Print first field
awk '{print $NF}' file             → Print last field
awk -F: '{print $1}' /etc/passwd   → Custom delimiter (colon)
awk '/pattern/' file               → Print matching lines
awk '!/pattern/' file              → Print non-matching lines
awk '{sum += $1} END {print sum}'  → Sum column
awk 'NR==5' file                   → Print line 5
awk 'NR>=5 && NR<=10' file         → Print lines 5-10
awk '{print NR, $0}' file          → Print with line numbers
awk '$3 > 100' file                → Conditional print
```

---

## COMMON REGEX PATTERNS

```text
# Email
^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$

# IPv4 address (shape, not validity — matches 999.999.999.999 too)
^(\d{1,3}\.){3}\d{1,3}$

# IPv6 address (full form)
^([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$

# URL
https?://[\w.-]+(?:\.[\w\.-]+)+[\w\-\._~:/?#[\]@!\$&'\(\)\*\+,;=.]+

# MAC address
^([0-9A-Fa-f]{2}[:-]){5}[0-9A-Fa-f]{2}$

# Date (YYYY-MM-DD)
^\d{4}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$

# Time (HH:MM:SS)
^([01]\d|2[0-3]):[0-5]\d:[0-5]\d$

# US phone number
^(\+1[-.\s]?)?(\(?\d{3}\)?[-.\s]?)?\d{3}[-.\s]?\d{4}$

# Zip code (US)
^\d{5}(-\d{4})?$

# Hex color
^#([0-9a-fA-F]{3}|[0-9a-fA-F]{6})$

# JWT token (three base64url segments)
^[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+\.[A-Za-z0-9_-]+$

# Ethereum address
^0x[0-9a-fA-F]{40}$

# Bitcoin address (simplified)
^[13][a-km-zA-HJ-NP-Z1-9]{25,34}$

# Solana public key (base58, 32-44 chars)
^[1-9A-HJ-NP-Za-km-z]{32,44}$

# IP in log line (extract)
\b(?:\d{1,3}\.){3}\d{1,3}\b

# Extract domain from URL
(?:https?://)?(?:www\.)?([^/\s]+)

# Find lines with ERROR in logs
^.*ERROR.*$

# Match log timestamp (ISO 8601)
^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}
```

---

## TESTING TOOLS

```text
https://regex101.com     → Best overall (explanation + test, per-flavor)
https://regexr.com       → Visual breakdown
https://regexper.com     → Railroad diagram visualization
```

```bash
echo "test123" | grep -P '\d+'
echo "test123" | sed 's/[0-9]//g'
python3 -c "import re; print(re.findall(r'\d+', 'test123'))"
```

---

## TIPS & GOTCHAS

- **Set the flavor on regex101 before trusting it** — a pattern validated as PCRE will fail in sed's default BRE. Match the tester to the tool.
- **`\d` fails silently in grep/sed** — BRE and ERE don't know it; it matches a literal "d" or errors. Use `[0-9]`, or `grep -P`.
- **Catastrophic backtracking** — nested quantifiers like `(a+)+` or `(.*)*` against non-matching input can hang for minutes. If a regex "freezes," this is why; make the inner pattern possessive-shaped or more specific.
- **Escape the dot in domains and IPs** — `192.168.1.1` without `\.` happily matches `192x168y1z1`. The IP patterns above do this correctly; homemade ones usually don't.
- **Shape ≠ validity** — the IPv4/date/phone patterns match well-formed *strings*; they don't range-check octets or validate calendars. Validate semantics in code, match shape with regex.
- **Always raw strings in Python** — `r'\d+'`, never `'\d+'`. Without `r`, Python's own escaping fights the regex's, and `'\b'` becomes a backspace character.
- **Quote patterns in the shell** — single quotes, always: `grep '\$[0-9]+'`. Unquoted `$`, `*`, and `\` are consumed by bash before grep sees them.
- **Anchor when you mean it** — `grep '404' access.log` also matches `14045` in a byte count. `\b404\b` or field-based awk (`$9 == 404`) says what you mean.

---
*Last Updated: 2026-07*
