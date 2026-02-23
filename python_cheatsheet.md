# python_cheatsheet

# DATA TYPES

---

x = 10              → Integer\
x = 3.14            → Float\
x = "hello"         → String\
x = True / False    → Boolean\
x = None            → NoneType\
x = \[1, 2, 3\]       → List (mutable, ordered)\
x = (1, 2, 3)       → Tuple (immutable, ordered)\
x = {1, 2, 3}       → Set (unique, unordered)\
x = {"k": "v"}      → Dictionary (key-value pairs)

---

## VARIABLES & TYPE CHECKING

x = 42                     → Assign variable\
type(x)                    → Return type of variable\
isinstance(x, int)         → Check if x is an int (returns bool)\
int("5") / str(5)          → Type casting\
f"{variable}"              → f-string (preferred string formatting)

---

## STRING OPERATIONS

s = "hello world"\
s.upper()                  → "HELLO WORLD"\
s.lower()                  → "hello world"\
s.strip()                  → Remove leading/trailing whitespace\
s.split(" ")               → Split into list by delimiter\
s.replace("hello", "hi")  → Replace substring\
s.startswith("hello")      → Returns True/False\
s.find("world")            → Returns index of substring (-1 if not found)\
len(s)                     → Length of string\
s\[0:5\]                     → Slice string (index 0 to 4)\
" ".join(\["a", "b", "c"\]) → Join list into string → "a b c"

---

## LIST OPERATIONS

lst = \[1, 2, 3\]\
lst.append(4)              → Add item to end\
lst.insert(1, 99)          → Insert 99 at index 1\
lst.remove(2)              → Remove first occurrence of value\
lst.pop()                  → Remove & return last item\
lst.pop(0)                 → Remove & return item at index\
lst.sort()                 → Sort in place (ascending)\
lst.reverse()              → Reverse list in place\
lst.index(3)               → Return index of value\
len(lst)                   → Length of list\
lst\[0\]                     → Access by index\
lst\[-1\]                    → Access last item\
lst\[1:3\]                   → Slice list\
\[x for x in lst if x > 1\] → List comprehension with filter

---

## DICTIONARY OPERATIONS

d = {"name": "Alice", "age": 30}\
d\["name"\]                  → Access value by key\
d.get("name", "default")  → Safe access (no KeyError)\
d\["city"\] = "NYC"          → Add/update key\
d.pop("age")               → Remove key & return value\
d.keys()                   → Return all keys\
d.values()                 → Return all values\
d.items()                  → Return all key-value pairs\
"name" in d                → Check if key exists (True/False)

---

## CONTROL FLOW

if x > 10:\
pass\
elif x == 10:\
pass\
else:\
pass

for i in range(5):         → Loop 0 to 4\
pass

for i, v in enumerate(lst):  → Loop with index\
pass

while condition:\
break                  → Exit loop\
continue               → Skip to next iteration

---

## FUNCTIONS

def greet(name, greeting="Hello"):   → Default parameter\
return f"{greeting}, {name}!"

lambda x: x \* 2            → Anonymous function\
\*args                       → Accept variable positional args\
\*\*kwargs                    → Accept variable keyword args

---

## ERROR HANDLING

try:\
risky_code()\
except ValueError as e:\
print(e)\
except (TypeError, KeyError):\
pass\
finally:\
pass                   → Always runs

raise ValueError("msg")    → Manually raise exception

---

## FILE I/O

with open("file.txt", "r") as f:    → Read\
content = f.read()\
lines = f.readlines()

with open("file.txt", "w") as f:    → Write (overwrites)\
f.write("hello\\n")

with open("file.txt", "a") as f:    → Append\
f.write("more\\n")

---

## CLASSES & OOP

class Dog:\
def **init**(self, name):        → Constructor\
[self.name](http://self.name/) = name

```
def bark(self):
    return f"{self.name} says woof!"
```

class Poodle(Dog):                   → Inheritance\
pass

isinstance(obj, Dog)                 → Check class membership

---

## USEFUL BUILT-INS

range(start, stop, step)   → Generate number sequence\
zip(lst1, lst2)            → Pair up two iterables\
map(func, lst)             → Apply function to each item\
filter(func, lst)          → Filter items by function\
sorted(lst, reverse=True)  → Return sorted copy\
min(lst) / max(lst)        → Min/max of iterable\
sum(lst)                   → Sum of iterable\
abs(x)                     → Absolute value\
round(x, 2)                → Round to decimal places\
print(x, end="", sep=",")  → Print with custom end/separator

---

## IMPORTS & MODULES

import os\
import sys\
from pathlib import Path\
import json\
import re                  → Regex\
from datetime import datetime\
import requests            → HTTP (pip install)

---

## COMMON PATTERNS

# Read JSON file

with open("data.json") as f:\
data = json.load(f)

# Write JSON file

with open("data.json", "w") as f:\
json.dump(data, f, indent=2)

# List all files in directory

files = os.listdir(".")

# Check if file/path exists

Path("file.txt").exists()

# Get current timestamp

datetime.now().strftime("%Y-%m-%d %H:%M:%S")

# Regex match

re.findall(r"\\d+", "abc123")   → \['123'\]