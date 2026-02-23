# bash_scripting_cheatsheet

# SCRIPT BASICS

---

#!/bin/bash                        → Shebang (always first line)
chmod +x [script.sh](http://script.sh/)                 → Make executable
./script.sh                        → Run script
bash [script.sh](http://script.sh/)                     → Run without executable bit
bash -x [script.sh](http://script.sh/)                  → Debug mode (print each command)
set -e                             → Exit on any error
set -u                             → Exit on undefined variable
set -o pipefail                    → Catch errors in pipes
set -euo pipefail                  → Recommended: use all three

---

## VARIABLES

name="Alice"                       → Assign (no spaces around =)
echo $name                         → Use variable
echo "${name}"                     → Safe variable syntax (preferred)
readonly PI=3.14                   → Constant
unset name                         → Delete variable
MY_VAR=$(command)                  → Capture command output
MY_VAR=$(cat file.txt)             → Capture file contents

---

## SPECIAL VARIABLES

$0        → Script name
$1 $2 ... → Positional arguments
$@        → All arguments (as separate words)
$#        → Number of arguments
$?        → Exit code of last command (0=success)
$$        → PID of current script
$!        → PID of last background process
$_        → Last argument of previous command

---

## USER INPUT

read -p "Enter name: " name        → Prompt and read input
read -s -p "Password: " pass       → Silent input (no echo)
read -t 10 -p "Timeout in 10s: " val  → Timeout read

---

## CONDITIONALS

if [ "$x" -eq 10 ]; then
echo "ten"
elif [ "$x" -gt 10 ]; then
echo "greater"
else
echo "less"
fi

# Double brackets (preferred for strings)

if [[ "$name" == "Alice" ]]; then ...
if [[ "$name" =~ ^Al ]]; then ...   → Regex match

# File tests

if [ -f file.txt ]; then ...        → File exists
if [ -d /path ]; then ...           → Directory exists
if [ -r file ]; then ...            → File is readable
if [ -z "$var" ]; then ...          → Variable is empty
if [ -n "$var" ]; then ...          → Variable is not empty

# Numeric comparisons

- eq equal
-ne not equal
-lt less than
-le less than or equal
-gt greater than
-ge greater than or equal

---

## LOOPS

# For loop

for i in 1 2 3; do echo $i; done
for i in {1..10}; do echo $i; done
for f in *.log; do echo "$f"; done
for i in $(seq 1 5); do echo $i; done

# C-style for loop

for ((i=0; i<5; i++)); do echo $i; done

# While loop

while [ $i -lt 10 ]; do
((i++))
done

# Read file line by line

while IFS= read -r line; do
echo "$line"
done < file.txt

# Until loop

until [ $i -ge 10 ]; do ((i++)); done

break     → Exit loop
continue  → Skip to next iteration

---

## FUNCTIONS

greet() {
local name="$1"                  → local scopes variable to function
echo "Hello, $name"
return 0                         → Return exit code (not value)
}
greet "Alice"
result=$(greet "Alice")            → Capture function output

---

## ARRAYS

arr=("a" "b" "c")                  → Define array
echo "${arr[0]}"                   → First element
echo "${arr[@]}"                   → All elements
echo "${#arr[@]}"                  → Length
arr+=("d")                         → Append element
for item in "${arr[@]}"; do ...    → Loop array

# Associative array (dict)

declare -A map
map["key"]="value"
echo "${map["key"]}"
for k in "${!map[@]}"; do echo "$k=${map[$k]}"; done

---

## STRING OPERATIONS

str="hello world"
echo "${#str}"                     → Length
echo "${str^^}"                    → Uppercase
echo "${str,,}"                    → Lowercase
echo "${str/hello/hi}"             → Replace first
echo "${str//l/L}"                 → Replace all
echo "${str:0:5}"                  → Substring (start:length)
echo "${str#hello }"               → Strip prefix
echo "${str%world}"                → Strip suffix
[[ "$str" == *"world"* ]]          → Contains check

---

## ARITHMETIC

result=$((5 + 3))                  → Integer arithmetic
((count++))                        → Increment
((count--))                        → Decrement
echo $((10 % 3))                   → Modulo
result=$(echo "3.14 * 2" | bc)     → Float math (requires bc)
result=$(echo "scale=2; 10/3" | bc)  → Float with precision

---

## OUTPUT & REDIRECTION

echo "text"                        → Print with newline
printf "%-10s %5d\n" "item" 42     → Formatted output
command > file.txt                 → Redirect stdout (overwrite)
command >> file.txt                → Redirect stdout (append)
command 2> error.log               → Redirect stderr
command 2>&1                       → Merge stderr into stdout
command &> file.txt                → Redirect both stdout & stderr
command | tee file.txt             → Print AND write to file
/dev/null                          → Discard output (black hole)

---

## ERROR HANDLING

command || echo "failed"           → Run if command fails
command && echo "success"          → Run if command succeeds
command || exit 1                  → Exit script on failure

trap 'echo "Error on line $LINENO"' ERR   → Catch errors
trap 'rm -f /tmp/tmpfile' EXIT            → Cleanup on exit

---

## COMMON PATTERNS

# Check if running as root

if [[ $EUID -ne 0 ]]; then
echo "Must run as root" && exit 1
fi

# Check if argument provided

if [[ $# -lt 1 ]]; then
echo "Usage: $0 <arg>" && exit 1
fi

# Timestamp

ts=$(date +"%Y-%m-%d_%H-%M-%S")

# Random temp file

tmpfile=$(mktemp /tmp/script.XXXXXX)

# Retry loop

for i in {1..3}; do
command && break || sleep 2
done

# Parse flags

while getopts "n:v" opt; do
case $opt in
n) name="$OPTARG" ;;
v) verbose=true ;;
esac
done

# Logging function

log() { echo "[$(date +%H:%M:%S)] $*"; }
log "Script started"