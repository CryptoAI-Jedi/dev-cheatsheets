# regex_cheatsheet

# CORE SYNTAX

---

.          → Any single character (except newline)
^          → Start of string / line
$          → End of string / line
\          → Escape special character  e.g. \. matches literal dot
|          → OR  e.g. cat|dog
()         → Capture group
(?:...)    → Non-capturing group
[]         → Character class  e.g. [aeiou]
[^]        → Negated character class  e.g. [^0-9]

---

## QUANTIFIERS

- `→ 0 or more`
- `→ 1 or more`

?          → 0 or 1 (optional)
{n}        → Exactly n times
{n,}       → n or more times
{n,m}      → Between n and m times

# Greedy vs Lazy

.*         → Greedy (match as much as possible)
.*?        → Lazy (match as little as possible)
.+?        → Lazy one or more

---

## CHARACTER CLASSES

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

# Custom classes

[a-z]      → Lowercase letter
[A-Z]      → Uppercase letter
[0-9]      → Digit
[a-zA-Z]   → Any letter
[a-zA-Z0-9_]  → Word characters (same as \w)
[^aeiou]   → Not a vowel

---

## ANCHORS & BOUNDARIES

^hello        → String starts with "hello"
world$        → String ends with "world"
^hello$       → Exact match "hello" only
\bword\b      → Whole word "word" (not "words" or "keyword")
\Bword\B      → "word" NOT at a word boundary

---

## GROUPS & REFERENCES

(abc)         → Capture group 1
(abc)(def)    → Groups 1 and 2
\1            → Backreference to group 1
(?P<name>...) → Named group (Python)
(?<name>...)  → Named group (JS/PCRE)
(?P=name)     → Backreference to named group (Python)
(?:abc)       → Non-capturing group (match but don't capture)
(?=abc)       → Positive lookahead (followed by abc)
(?!abc)       → Negative lookahead (NOT followed by abc)
(?<=abc)      → Positive lookbehind (preceded by abc)
(?<!abc)      → Negative lookbehind (NOT preceded by abc)

---

## FLAGS / MODIFIERS

i     → Case-insensitive  /pattern/i
g     → Global (find all matches)  /pattern/g
m     → Multiline (^ and $ match line boundaries)
s     → Dotall (. matches newline too)
x     → Extended (allow whitespace/comments in pattern)

# In Python

re.IGNORECASE  (re.I)
re.MULTILINE   (re.M)
re.DOTALL      (re.S)
re.VERBOSE     (re.X)

---

## PYTHON (re module)

import re

re.search(r'\d+', text)            → First match anywhere in string
re.match(r'\d+', text)             → Match at START of string only
re.fullmatch(r'\d+', text)         → Entire string must match
re.findall(r'\d+', text)           → List of all matches
re.finditer(r'\d+', text)          → Iterator of match objects
re.sub(r'\d+', 'NUM', text)        → Replace all matches
re.sub(r'\d+', 'NUM', text, count=1)  → Replace first only
re.split(r'\s+', text)             → Split on whitespace
re.compile(r'\d+')                 → Compile for reuse (faster in loops)

# Match object methods

m = re.search(r'(\d+)', text)
m.group()          → Full match
m.group(1)         → Capture group 1
m.groups()         → All capture groups as tuple
m.start() / m.end() → Match position
m.span()           → (start, end) tuple

# Named groups (Python)

m = re.search(r'(?P<year>\d{4})-(?P<month>\d{2})', text)
m.group('year')
m.group('month')

---

## GREP USAGE

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

---

## SED USAGE

sed 's/old/new/' file              → Replace first on each line
sed 's/old/new/g' file             → Replace all occurrences
sed 's/old/new/gi' file            → Case-insensitive replace all
sed -i 's/old/new/g' file          → In-place edit
sed -i.bak 's/old/new/g' file      → In-place with backup
sed -n '5,10p' file                → Print lines 5-10
sed '5,10d' file                   → Delete lines 5-10
sed '/pattern/d' file              → Delete lines matching pattern
sed -n '/pattern/p' file           → Print only matching lines
sed 's/\b[0-9]\{3\}\b/NUM/g' file → Replace 3-digit numbers

---

## AWK USAGE

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

---

## COMMON REGEX PATTERNS

# Email

^[\w.+-]+@[\w-]+\.[a-zA-Z]{2,}$

# IPv4 address

^(\d{1,3}\.){3}\d{1,3}$

# IPv6 address

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

# JWT token

^[A-Za-z0-9-*]+\.[A-Za-z0-9-*]+\.[A-Za-z0-9-_]+$

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

# Match log timestamp

^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}

---

## TESTING TOOLS

# Online

[https://regex101.com](https://regex101.com/)               → Best overall (explain + test)
[https://regexr.com](https://regexr.com/)                 → Visual breakdown
[https://regexper.com](https://regexper.com/)               → Railroad diagram visualization

# CLI

echo "test123" | grep -P '\d+'
echo "test123" | sed 's/[0-9]//g'
python3 -c "import re; print(re.findall(r'\d+', 'test123'))"