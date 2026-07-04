# Python Cheatsheet

> Core language syntax: types, strings, collections, control flow, functions, classes, and the patterns that show up in every script. Environments and packaging live in the [Python Environments & Packaging sheet](python_environments_packaging_cheatsheet.md).

---

## Table of Contents
- [Data Types](#data-types)
- [Variables & Type Checking](#variables--type-checking)
- [String Operations](#string-operations)
- [f-strings](#f-strings)
- [List Operations](#list-operations)
- [Dictionary Operations](#dictionary-operations)
- [Comprehensions](#comprehensions)
- [Control Flow](#control-flow)
- [Functions](#functions)
- [Error Handling](#error-handling)
- [File I/O](#file-io)
- [pathlib](#pathlib)
- [Classes & OOP](#classes--oop)
- [Useful Built-ins](#useful-built-ins)
- [Imports & Modules](#imports--modules)
- [Common Patterns](#common-patterns)
- [Tips & Gotchas](#tips--gotchas)

---

## DATA TYPES

```python
x = 10              # Integer
x = 3.14            # Float
x = "hello"         # String
x = True            # Boolean (True / False)
x = None            # NoneType
x = [1, 2, 3]       # List (mutable, ordered)
x = (1, 2, 3)       # Tuple (immutable, ordered)
x = {1, 2, 3}       # Set (unique, unordered)
x = {"k": "v"}      # Dictionary (key-value pairs)
```

---

## VARIABLES & TYPE CHECKING

```python
x = 42                      # Assign variable
type(x)                     # Return type of variable
isinstance(x, int)          # Check if x is an int (returns bool)
int("5"); str(5); float(3)  # Type casting
f"{variable}"               # f-string (preferred string formatting)
```

---

## STRING OPERATIONS

```python
s = "hello world"
s.upper()                   # "HELLO WORLD"
s.lower()                   # "hello world"
s.strip()                   # Remove leading/trailing whitespace
s.split(" ")                # Split into list by delimiter
s.replace("hello", "hi")    # Replace substring
s.startswith("hello")       # Returns True/False
s.endswith("world")         # Returns True/False
s.find("world")             # Index of substring (-1 if not found)
len(s)                      # Length of string
s[0:5]                      # Slice string (index 0 to 4)
s[::-1]                     # Reverse a string
" ".join(["a", "b", "c"])   # Join list into string → "a b c"
```

---

## F-STRINGS

```python
f"{name} is {age}"            # Interpolation
f"{price:.2f}"                # 2 decimal places → "3.14"
f"{n:,}"                      # Thousands separator → "1,234,567"
f"{ratio:.1%}"                # Percentage → "42.0%"
f"{text:>10}"                 # Right-align in 10 chars (< left, ^ center)
f"{value=}"                   # Debug form → "value=42"
f"{ts:%Y-%m-%d}"              # Format datetimes inline
```

---

## LIST OPERATIONS

```python
lst = [1, 2, 3]
lst.append(4)               # Add item to end
lst.insert(1, 99)           # Insert 99 at index 1
lst.remove(2)               # Remove first occurrence of value
lst.pop()                   # Remove & return last item
lst.pop(0)                  # Remove & return item at index
lst.sort()                  # Sort in place (ascending)
lst.sort(key=len)           # Sort by a key function
lst.reverse()               # Reverse list in place
lst.index(3)                # Return index of value
len(lst)                    # Length of list
lst[0]                      # Access by index
lst[-1]                     # Access last item
lst[1:3]                    # Slice list
a, b, *rest = lst           # Unpacking
```

---

## DICTIONARY OPERATIONS

```python
d = {"name": "Alice", "age": 30}
d["name"]                   # Access value by key (KeyError if missing)
d.get("name", "default")    # Safe access (no KeyError)
d["city"] = "NYC"           # Add/update key
d.pop("age")                # Remove key & return value
d.keys()                    # All keys
d.values()                  # All values
d.items()                   # All key-value pairs
"name" in d                 # Check if key exists (True/False)
d.setdefault("tags", [])    # Get key, inserting default if absent
merged = d1 | d2            # Merge dicts (3.9+; right side wins)
for k, v in d.items():      # Iterate pairs
    ...
```

---

## COMPREHENSIONS

```python
[x * 2 for x in lst]                  # List comprehension
[x for x in lst if x > 1]             # With filter
{x: x**2 for x in range(5)}           # Dict comprehension
{x % 3 for x in lst}                  # Set comprehension
(x * 2 for x in lst)                  # Generator (lazy — no list in memory)
sum(x * 2 for x in lst)               # Feed a generator straight to a function
```

---

## CONTROL FLOW

```python
if x > 10:
    ...
elif x == 10:
    ...
else:
    ...

for i in range(5):                    # Loop 0 to 4
    ...

for i, v in enumerate(lst):           # Loop with index
    ...

for a, b in zip(lst1, lst2):          # Loop two lists in lockstep
    ...

while condition:
    if done:
        break                         # Exit loop
    if skip:
        continue                      # Skip to next iteration
```

---

## FUNCTIONS

```python
def greet(name, greeting="Hello"):    # Default parameter
    return f"{greeting}, {name}!"

def f(*args, **kwargs):               # Variable positional / keyword args
    ...

greet("Alice", greeting="Hey")        # Keyword arguments at call site

double = lambda x: x * 2              # Anonymous function
```

---

## ERROR HANDLING

```python
try:
    risky_code()
except ValueError as e:
    print(e)
except (TypeError, KeyError):
    ...
else:
    ...                               # Runs only if NO exception
finally:
    ...                               # Always runs

raise ValueError("msg")               # Manually raise exception
raise                                 # Re-raise current exception (in except block)
```

---

## FILE I/O

```python
with open("file.txt", "r") as f:      # Read
    content = f.read()
    # or: lines = f.readlines()

with open("file.txt", "w") as f:      # Write (overwrites)
    f.write("hello\n")

with open("file.txt", "a") as f:      # Append
    f.write("more\n")

with open("file.txt") as f:           # Memory-friendly line iteration
    for line in f:
        process(line.rstrip())
```

---

## PATHLIB

```python
from pathlib import Path

p = Path("data") / "config.json"      # Paths compose with /
p.exists(); p.is_file(); p.is_dir()
p.read_text()                         # Whole file as string (no open() dance)
p.write_text("content")
p.name; p.stem; p.suffix; p.parent    # "config.json", "config", ".json", data/
Path.home() / ".config"
p.mkdir(parents=True, exist_ok=True)
list(Path(".").glob("*.md"))          # Glob files
list(Path(".").rglob("*.py"))         # Recursive glob
```

---

## CLASSES & OOP

```python
class Dog:
    def __init__(self, name):         # Constructor
        self.name = name

    def bark(self):
        return f"{self.name} says woof!"

class Poodle(Dog):                    # Inheritance
    def __init__(self, name, size):
        super().__init__(name)        # Call parent constructor
        self.size = size

d = Dog("Rex")
d.bark()
isinstance(d, Dog)                    # Check class membership
```

```python
from dataclasses import dataclass

@dataclass                            # Auto __init__, __repr__, __eq__
class Point:
    x: int
    y: int = 0
```

---

## USEFUL BUILT-INS

```python
range(start, stop, step)    # Generate number sequence
zip(lst1, lst2)             # Pair up two iterables
map(func, lst)              # Apply function to each item
filter(func, lst)           # Filter items by function
sorted(lst, reverse=True)   # Return sorted copy
sorted(lst, key=lambda x: x.age)   # Sort by key
min(lst); max(lst)          # Min/max of iterable
sum(lst)                    # Sum of iterable
any(x > 5 for x in lst)     # True if any pass
all(x > 0 for x in lst)     # True if all pass
abs(x)                      # Absolute value
round(x, 2)                 # Round to decimal places
print(x, end="", sep=",")   # Print with custom end/separator
```

---

## IMPORTS & MODULES

```python
import os
import sys
import json
import re                             # Regex (see regex sheet)
from pathlib import Path
from datetime import datetime, timedelta
import httpx                          # HTTP (pip install — see env sheet)
```

---

## COMMON PATTERNS

```python
# Script entry point — code that runs only when executed, not imported:
def main():
    ...

if __name__ == "__main__":
    main()
```

```python
# Read JSON file
with open("data.json") as f:
    data = json.load(f)

# Write JSON file
with open("data.json", "w") as f:
    json.dump(data, f, indent=2)

# JSON ↔ string (API payloads)
payload = json.dumps({"k": "v"})
data = json.loads(response_text)

# List all files in directory
files = os.listdir(".")               # or: list(Path(".").iterdir())

# Check if file/path exists
Path("file.txt").exists()

# Current timestamp
datetime.now().strftime("%Y-%m-%d %H:%M:%S")

# Parse a timestamp string
dt = datetime.strptime("2026-07-03", "%Y-%m-%d")

# Time math
yesterday = datetime.now() - timedelta(days=1)

# Regex match (see regex sheet)
re.findall(r"\d+", "abc123")          # ['123']
```

---

## TIPS & GOTCHAS

- **Mutable default arguments** — `def f(items=[])` shares ONE list across every call; it accumulates. Use `def f(items=None): items = items or []`.
- **`is` vs `==`** — `==` compares values, `is` compares identity. Only use `is` for `None`: `if x is None`.
- **`/` vs `//`** — `/` always returns float (`10/2` → `5.0`); `//` is floor division (`7//2` → `3`).
- **Copies are shallow by default** — `b = a` is the SAME list; `b = a.copy()` copies one level; nested structures need `copy.deepcopy(a)`.
- **Never modify a list while iterating it** — skipped elements and silent bugs. Iterate a copy (`for x in lst[:]`) or build a new list via comprehension.
- **`sort()` returns None** — `lst = lst.sort()` destroys your list. In-place: `lst.sort()`; new list: `sorted(lst)`.
- **`except:` bare catches everything** — including Ctrl+C and typos. Catch the narrowest exception you can name; `except Exception` at minimum.
- **String building in loops** — `s += piece` re-copies the string every pass. Collect in a list, `"".join(parts)` once.

---
*Last Updated: 2026-07*
