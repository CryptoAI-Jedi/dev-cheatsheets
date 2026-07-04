# Bash Scripting Cheatsheet

> Shell scripting for automation: variables, conditionals, loops, functions, and the safety rails (`set -euo pipefail`, shellcheck, quoting) that keep scripts from eating servers.

---

## Table of Contents
- [Script Basics](#script-basics)
- [Variables](#variables)
- [Special Variables](#special-variables)
- [User Input](#user-input)
- [Conditionals](#conditionals)
- [case Statement](#case-statement)
- [Loops](#loops)
- [Functions](#functions)
- [Arrays](#arrays)
- [String Operations](#string-operations)
- [Arithmetic](#arithmetic)
- [Output & Redirection](#output--redirection)
- [Here-docs & Here-strings](#here-docs--here-strings)
- [Error Handling](#error-handling)
- [Common Patterns](#common-patterns)
- [Tips & Gotchas](#tips--gotchas)

---

## SCRIPT BASICS

```bash
#!/bin/bash                        # Shebang (always first line)
chmod +x script.sh                 # Make executable
./script.sh                        # Run script
bash script.sh                     # Run without executable bit
bash -x script.sh                  # Debug mode (print each command)
bash -n script.sh                  # Syntax check without running
set -e                             # Exit on any error
set -u                             # Exit on undefined variable
set -o pipefail                    # Catch errors in pipes
set -euo pipefail                  # Recommended: all three, top of every script
```

```bash
sudo apt install shellcheck
shellcheck script.sh               # Lint — catches quoting/logic bugs before prod does
```

---

## VARIABLES

```bash
name="Alice"                       # Assign (NO spaces around =)
echo "$name"                       # Use variable (quoted — see Gotchas)
echo "${name}"                     # Brace syntax (required adjacent to text: "${name}_log")
readonly PI=3.14                   # Constant
unset name                         # Delete variable
MY_VAR=$(command)                  # Capture command output
MY_VAR=$(cat file.txt)             # Capture file contents
: "${PORT:=8080}"                  # Default if unset (assigns)
echo "${PORT:-8080}"               # Default if unset (doesn't assign)
```

---

## SPECIAL VARIABLES

```text
$0        → Script name
$1 $2 ... → Positional arguments
$@        → All arguments (as separate words — almost always what you want)
$*        → All arguments (as one word — rarely what you want)
$#        → Number of arguments
$?        → Exit code of last command (0 = success)
$$        → PID of current script
$!        → PID of last background process
$_        → Last argument of previous command
```

---

## USER INPUT

```bash
read -p "Enter name: " name           # Prompt and read input
read -s -p "Password: " pass          # Silent input (no echo)
read -t 10 -p "Timeout in 10s: " val  # Timeout read
read -r line                          # -r: don't mangle backslashes (default to this)
```

---

## CONDITIONALS

```bash
if [ "$x" -eq 10 ]; then
    echo "ten"
elif [ "$x" -gt 10 ]; then
    echo "greater"
else
    echo "less"
fi
```

### Double brackets (preferred for strings)

```bash
if [[ "$name" == "Alice" ]]; then ...
if [[ "$name" == Al* ]]; then ...      # Glob match
if [[ "$name" =~ ^Al ]]; then ...      # Regex match
if [[ -n "$a" && -z "$b" ]]; then ...  # && / || work inside [[ ]]
```

### File tests

```bash
if [ -f file.txt ]; then ...        # File exists
if [ -d /path ]; then ...           # Directory exists
if [ -r file ]; then ...            # File is readable
if [ -x file ]; then ...            # File is executable
if [ -s file ]; then ...            # File exists AND is non-empty
if [ -z "$var" ]; then ...          # Variable is empty
if [ -n "$var" ]; then ...          # Variable is not empty
```

### Numeric comparisons

```text
-eq   equal
-ne   not equal
-lt   less than
-le   less than or equal
-gt   greater than
-ge   greater than or equal
```

---

## CASE STATEMENT

```bash
case "$1" in
    start)
        echo "starting" ;;
    stop|halt)                      # Multiple patterns
        echo "stopping" ;;
    -v|--verbose)
        verbose=true ;;
    *)                              # Default
        echo "Usage: $0 {start|stop}" && exit 1 ;;
esac
```

---

## LOOPS

```bash
# For loop
for i in 1 2 3; do echo "$i"; done
for i in {1..10}; do echo "$i"; done
for f in *.log; do echo "$f"; done
for i in $(seq 1 5); do echo "$i"; done

# C-style for loop
for ((i=0; i<5; i++)); do echo "$i"; done

# While loop
while [ "$i" -lt 10 ]; do
    ((i++))
done

# Read file line by line (the ONLY safe way)
while IFS= read -r line; do
    echo "$line"
done < file.txt

# Until loop
until [ "$i" -ge 10 ]; do ((i++)); done
```

```text
break     → Exit loop
continue  → Skip to next iteration
```

---

## FUNCTIONS

```bash
greet() {
    local name="$1"                # local scopes variable to function
    echo "Hello, $name"
    return 0                       # Return EXIT CODE (0-255), not a value
}

greet "Alice"
result=$(greet "Alice")            # "Return" values by capturing output
```

---

## ARRAYS

```bash
arr=("a" "b" "c")                  # Define array
echo "${arr[0]}"                   # First element
echo "${arr[@]}"                   # All elements
echo "${#arr[@]}"                  # Length
arr+=("d")                         # Append element
for item in "${arr[@]}"; do        # Loop array (quotes mandatory)
    echo "$item"
done
```

### Associative array (dict)

```bash
declare -A map
map["key"]="value"
echo "${map["key"]}"
for k in "${!map[@]}"; do echo "$k=${map[$k]}"; done
```

---

## STRING OPERATIONS

```bash
str="hello world"
echo "${#str}"                     # Length
echo "${str^^}"                    # Uppercase
echo "${str,,}"                    # Lowercase
echo "${str/hello/hi}"             # Replace first
echo "${str//l/L}"                 # Replace all
echo "${str:0:5}"                  # Substring (start:length)
echo "${str#hello }"               # Strip prefix
echo "${str%world}"                # Strip suffix
echo "${file%.log}"                # Strip extension → common use
echo "${path##*/}"                 # basename via expansion
[[ "$str" == *"world"* ]]          # Contains check
```

---

## ARITHMETIC

```bash
result=$((5 + 3))                  # Integer arithmetic
((count++))                        # Increment
((count--))                        # Decrement
echo $((10 % 3))                   # Modulo
result=$(echo "3.14 * 2" | bc)     # Float math (requires bc)
result=$(echo "scale=2; 10/3" | bc)  # Float with precision
```

---

## OUTPUT & REDIRECTION

```bash
echo "text"                        # Print with newline
printf "%-10s %5d\n" "item" 42     # Formatted output
command > file.txt                 # Redirect stdout (overwrite)
command >> file.txt                # Redirect stdout (append)
command 2> error.log               # Redirect stderr
command 2>&1                       # Merge stderr into stdout
command &> file.txt                # Redirect both stdout & stderr
command > /dev/null 2>&1           # Discard everything
command | tee file.txt             # Print AND write to file
command | tee -a file.txt          # Print AND append
```

---

## HERE-DOCS & HERE-STRINGS

```bash
cat << EOF > config.txt            # Multi-line content ($vars expand)
server=$HOSTNAME
port=8080
EOF

cat << 'EOF' > script.sh           # Quoted delimiter: NO expansion (literal $)
echo "$1"
EOF

sudo tee /etc/app.conf > /dev/null << EOF    # Heredoc into a root-owned file
key=value
EOF

grep "pattern" <<< "$variable"     # Here-string: feed a variable as stdin
```

---

## ERROR HANDLING

```bash
command || echo "failed"           # Run if command fails
command && echo "success"          # Run if command succeeds
command || exit 1                  # Exit script on failure

trap 'echo "Error on line $LINENO"' ERR    # Catch errors
trap 'rm -f "$tmpfile"' EXIT               # Cleanup on ANY exit (even Ctrl+C)
```

```bash
if ! command -v jq &> /dev/null; then      # Dependency check
    echo "jq required" && exit 1
fi
```

---

## COMMON PATTERNS

```bash
# Check if running as root
if [[ $EUID -ne 0 ]]; then
    echo "Must run as root" && exit 1
fi

# Check if argument provided
if [[ $# -lt 1 ]]; then
    echo "Usage: $0 <arg>" && exit 1
fi

# Resolve the script's own directory (works from anywhere)
SCRIPT_DIR=$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)

# Timestamp
ts=$(date +"%Y-%m-%d_%H-%M-%S")

# Random temp file / dir (cleanup with the EXIT trap above)
tmpfile=$(mktemp /tmp/script.XXXXXX)
tmpdir=$(mktemp -d)

# Retry loop
for i in {1..3}; do
    command && break || sleep 2
done

# Parse flags
while getopts "n:v" opt; do
    case $opt in
        n) name="$OPTARG" ;;
        v) verbose=true ;;
        *) exit 1 ;;
    esac
done

# Logging function
log() { echo "[$(date +%H:%M:%S)] $*"; }
log "Script started"
```

---

## TIPS & GOTCHAS

- **Quote every variable expansion** — `rm $file` with `file="my report.txt"` deletes `my` and `report.txt`. `"$file"` always; shellcheck will nag you until it's habit.
- **`[` vs `[[`** — `[` is a POSIX command with word-splitting landmines; `[[` is bash syntax with safe strings, `&&`/`||`, globs, and regex. Inside bash scripts, default to `[[`.
- **Spaces around `=`** — `x = 5` runs a command named `x`; `x=5` assigns. Inverse in tests: `[[ "$x"=="5" ]]` is always true (one word); `[[ "$x" == "5" ]]` compares.
- **`set -e` has blind spots** — failures inside `if` conditions, `||`/`&&` chains, and non-final pipeline stages don't trigger exit. `pipefail` fixes pipes; the rest is design.
- **`sh script.sh` is not bash on Ubuntu** — `sh` is dash; arrays, `[[`, and `${var//}` all break with confusing errors. Run bash scripts with `bash` or the shebang.
- **`^M: bad interpreter`** — Windows CRLF line endings. Fix: `sed -i 's/\r$//' script.sh` (or `dos2unix`).
- **`$@` vs `$*`, quoted** — `"$@"` preserves each argument as its own word (pass-through done right); `"$*"` mashes them into one. Wrappers and loops want `"$@"`.
- **`((count++))` can kill `set -e` scripts** — when count is 0, the expression evaluates to 0 → exit code 1 → script dies. Use `((count+=1))` or `count=$((count+1))`.
- **Run shellcheck before production, every time** — it catches 90% of the above mechanically. It's in apt; there's no excuse.

---
*Last Updated: 2026-07*
