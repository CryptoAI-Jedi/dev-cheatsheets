# jq Cheatsheet

> sed/awk/grep for JSON. Extract, filter, transform, and reshape JSON from APIs, logs, and RPC endpoints. The other half of every curl pipeline.

---

## Table of Contents
- [Basic Usage](#basic-usage)
- [Basic Field Access](#basic-field-access)
- [Nested Access](#nested-access)
- [Arrays](#arrays)
- [Transforming Output](#transforming-output)
- [Filtering Arrays (select)](#filtering-arrays-select)
- [Shell Variables into jq](#shell-variables-into-jq)
- [String Operations](#string-operations)
- [Numbers & Math](#numbers--math)
- [Dates](#dates)
- [Array Operations](#array-operations)
- [Object Operations](#object-operations)
- [Conditionals](#conditionals)
- [Variables & Reduce](#variables--reduce)
- [Combining Filters](#combining-filters)
- [Recursive Descent & Optional Access](#recursive-descent--optional-access)
- [Paths](#paths)
- [Type Functions](#type-functions)
- [Multiple Files & Slurp](#multiple-files--slurp)
- [Input / Output Formats](#input--output-formats)
- [Format Strings](#format-strings)
- [NDJSON & Log Streams](#ndjson--log-streams)
- [Practical Examples](#practical-examples)
- [Blockchain Examples](#blockchain-examples)
- [jq Flags Reference](#jq-flags-reference)
- [Debugging](#debugging)
- [Tips & Gotchas](#tips--gotchas)

---

## BASIC USAGE

```bash
sudo apt install jq        # Ubuntu/Debian (ships 1.6 — see Gotchas)
```

```text
cat file.json | jq '.'          → Pretty-print JSON
curl -s https://api.example.com | jq '.'
jq '.' file.json                → Same as above (file directly)
jq -r '.' file.json             → Raw output (no quotes on strings)
jq -c '.' file.json             → Compact output (one line)
jq -n '{"key": "value"}'        → Create JSON from scratch (null input)
jq '.' <<< '{"name":"Alice"}'   → Inline JSON input
```

---

## BASIC FIELD ACCESS

```text
# Given: {"name": "Alice", "age": 30}
jq '.name'                  → "Alice"
jq '.age'                   → 30
jq '.missing'               → null (no error)
jq '.missing // "default"'  → "default" (alternative operator)
jq -r '.name'               → Alice (raw, no quotes)
```

---

## NESTED ACCESS

```text
# Given: {"user": {"name": "Alice", "address": {"city": "KL"}}}
jq '.user.name'             → "Alice"
jq '.user.address.city'     → "KL"
jq '.user | .name'          → "Alice" (pipe within jq)
```

---

## ARRAYS

```text
# Given: [1, 2, 3, 4, 5]
jq '.[0]'      → 1 (first element)
jq '.[-1]'     → 5 (last element)
jq '.[2:4]'    → [3, 4] (slice)
jq '.[]'       → Explode array (each element on own line)
jq 'length'    → 5 (array length)
jq '.[0:3]'    → [1, 2, 3]
jq 'limit(3; .[])'  → First 3 elements as a stream
```

```text
# Array of objects
# Given: [{"name":"Alice","age":30},{"name":"Bob","age":25}]
jq '.[].name'    → "Alice" then "Bob"
jq '.[0].name'   → "Alice"
jq 'length'      → 2 (number of objects)
```

---

## TRANSFORMING OUTPUT

```text
jq '{username: .name, years: .age}'   → Build new object
jq '[.name, .age]'                    → Build array from fields
jq '[.[] | {n: .name, a: .age}]'      → Array of objects → new array of objects
jq '.[] | {name, age}'                → Shorthand (key = field name)
```

---

## FILTERING ARRAYS (select)

```text
# Given: [{"name":"Alice","age":30},{"name":"Bob","age":25}]
jq '.[] | select(.age > 27)'                 → Alice object only
jq '.[] | select(.name == "Bob")'            → Bob object only
jq '[.[] | select(.age >= 25)]'              → Array of matches
jq '.[] | select(.name | startswith("A"))'
jq '.[] | select(.role != null)'             → Has field
jq '.[] | select(.active == true)'
```

---

## SHELL VARIABLES INTO jq

Never interpolate shell variables into the filter string — pass them properly:

```bash
jq --arg name "Alice" '.[] | select(.name == $name)' users.json      # As string
jq --argjson min 25 '.[] | select(.age >= $min)' users.json          # As typed JSON
jq -n --arg host "$HOSTNAME" --argjson ok true '{host: $host, healthy: $ok}'   # Build payloads
```

---

## STRING OPERATIONS

```text
jq '.name | length'                    → String length
jq '.name | ascii_downcase'            → Lowercase
jq '.name | ascii_upcase'              → Uppercase
jq '.name | ltrimstr("prefix")'        → Remove prefix
jq '.name | rtrimstr("suffix")'        → Remove suffix
jq '.name | split(" ")'                → Split string → array
jq '.tags | join(", ")'                → Array → comma-separated string
jq '"\(.name) is \(.age) years old"'   → String interpolation
jq '.name | test("^A")'                → Regex test → true/false
jq '.name | match("^(A\\w+)")'         → Regex match object
jq '.name | capture("(?<first>\\w+)")' → Named capture → {"first": "..."}
jq '.msg | gsub("\\s+"; "_")'          → Regex find & replace
```

```text
⚠️ Regex backslashes are DOUBLED inside jq strings ("\\w+", "\\d") —
single \w is an invalid jq string escape and errors. See Gotchas.
```

---

## NUMBERS & MATH

```text
jq '.price * .qty'              → Multiply fields
jq '.price | floor'             → Floor
jq '.price | ceil'              → Ceiling
jq '.price | round'             → Round
jq '.price | fabs'              → Absolute value
jq '.price | sqrt'              → Square root
jq '[.[] | .price] | add'       → Sum all prices
jq '[.[] | .price] | min'       → Min
jq '[.[] | .price] | max'       → Max
jq '[.[] | .price] | add / length' → Average
jq 'tonumber'                   → "42" → 42
jq 'tostring'                   → 42 → "42"
```

---

## DATES

```text
jq 'now'                        → Unix epoch (float)
jq 'now | todate'               → "2026-07-03T18:00:00Z"
jq '.ts | todate'               → Unix timestamp → ISO 8601
jq '.time | fromdate'           → ISO 8601 → unix timestamp
jq 'now | gmtime | strftime("%Y-%m-%d %H:%M")'  → Custom format
```

---

## ARRAY OPERATIONS

```text
jq '[.[] | .price]'         → Extract field into array (map)
jq 'map(.price)'            → Same as above (cleaner)
jq 'map(. * 2)'             → Double every element
jq 'map(select(. > 10))'    → Filter array elements
jq 'map(.name | ascii_upcase)' → Transform each
jq 'sort'                   → Sort array of primitives
jq 'sort_by(.age)'          → Sort objects by field
jq 'sort_by(.age) | reverse' → Sort descending
jq 'unique'                 → Deduplicate array
jq 'unique_by(.name)'       → Deduplicate by field
jq 'group_by(.role)'        → Group objects by field
jq 'flatten'                → Flatten nested arrays
jq 'flatten(1)'             → Flatten one level
jq 'first'                  → First element
jq 'last'                   → Last element
jq 'nth(2)'                 → Third element (0-indexed)
jq 'reverse'                → Reverse array
jq 'indices("x")'           → Positions of "x"
jq 'contains([3])'          → Check if array contains 3
jq 'inside([1,2,3])'        → Inverse of contains
jq 'any(. > 2)'             → True if any element > 2
jq 'all(. > 0)'             → True if all elements > 0
jq 'add'                    → Sum / concatenate array
jq 'del(.[2])'              → Delete element at index 2
```

---

## OBJECT OPERATIONS

```text
jq 'keys'                       → Array of object keys
jq 'values'                     → Array of object values
jq 'keys_unsorted'              → Keys without sorting
jq 'to_entries'                 → [{key,value}] pairs
jq 'from_entries'               → [{key,value}] → object
jq 'with_entries(.value += 1)'  → Transform each value
jq 'has("name")'                → Check key exists
jq 'in({"a":1,"b":2})'          → Reverse has
jq '. + {"newkey": "val"}'      → Add/merge field
jq 'del(.unwanted)'             → Remove field
jq 'to_entries | map(select(.value != null)) | from_entries' → Remove null fields
```

---

## CONDITIONALS

```text
jq 'if .age > 18 then "adult" else "minor" end'
jq 'if .status == "active" then . else empty end'   → Filter with if
jq '.type // "unknown"'                             → Default if null
jq 'if . == null then "N/A" elif . == 0 then "zero" else . end'
```

---

## VARIABLES & REDUCE

```text
jq '. as $x | $x.name'                  → Bind to variable
jq '[.[] | . as $item | $item.name]'
```

```text
# reduce
jq 'reduce .[] as $x (0; . + $x)'        → Sum all numbers
jq 'reduce .[] as $x (0; . + $x.price)'  → Sum prices from objects
```

---

## COMBINING FILTERS

```text
jq '.[] | .name, .age'      → Multiple outputs per item
jq '.[] | [.name, .age]'    → As array per item
jq '(.a + .b) / 2'          → Expression grouping
```

---

## RECURSIVE DESCENT & OPTIONAL ACCESS

```text
jq '.. | .email? // empty'          → Find every "email" ANYWHERE in the tree
jq '[.. | objects | select(has("error"))]' → All objects containing an "error" key
jq '.foo?'                          → null instead of error on wrong type
jq '.[]?'                           → Iterate; silent if input isn't iterable
```

---

## PATHS

```text
jq 'path(.user.name)'                → ["user","name"]
jq 'getpath(["user","name"])'        → Value at path
jq 'setpath(["user","name"]; "Bob")' → Set value at path
jq 'delpaths([["user","age"]])'      → Delete paths
```

---

## TYPE FUNCTIONS

```text
jq 'type'       → "null","boolean","number","string","array","object"
jq 'numbers'    → Filter: only numbers
jq 'strings'    → Filter: only strings
jq 'booleans'   → Filter: only booleans
jq 'arrays'     → Filter: only arrays
jq 'objects'    → Filter: only objects
jq 'nulls'      → Filter: only nulls
jq 'scalars'    → Filter: non-iterables
jq 'iterables'  → Filter: arrays + objects
jq 'empty'      → Produce no output (useful for filtering)
jq 'fromjson'   → Parse a JSON string INSIDE a value
jq 'tojson'     → Serialize a value to a JSON string
```

---

## MULTIPLE FILES & SLURP

```text
jq -s '.' file1.json file2.json     → Slurp multiple files into array
jq -s 'add' file1.json file2.json   → Merge JSON objects from two files
jq -s 'flatten' *.json              → Combine all JSON arrays
```

---

## INPUT / OUTPUT FORMATS

```text
jq -r '.name'           → Raw string output (no quotes)
jq -j '.name'           → Raw output + no newline
jq -R '.'               → Raw input (treat input as string)
jq -Rs '.'              → Raw input + slurp into single string
jq -Rs 'split("\n")'    → Read lines into array
jq -r '.[]' file.json   → Print each element (no quotes)
```

---

## FORMAT STRINGS

```text
jq -r '.[] | [.name, .age, .city] | @csv'   → CSV format
jq -r '.[] | [.name, .age, .city] | @tsv'   → TSV format
jq -r '.[] | [.name, .age] | @csv' > out.csv
jq -r '.token | @base64'                    → Encode base64
jq -r '.blob | @base64d'                    → Decode base64
jq -r '.q | @uri'                           → URL-encode
jq -r '.arg | @sh'                          → Shell-quote (safe interpolation)
```

---

## NDJSON & LOG STREAMS

NDJSON (one JSON object per line) is the native shape of log pipelines — jq processes it record by record without slurping.

```bash
journalctl -u myagent -o json -n 50 | jq -r '.MESSAGE'                 # Journal → messages
journalctl -u myagent -o json | jq -r 'select(.PRIORITY == "3") | .MESSAGE'   # err+ only (fields are strings)
docker logs myapp 2>&1 | jq -r 'select(.level == "error") | .msg'      # JSON-logging apps
tail -f app.ndjson | jq -c 'select(.status >= 500)'                    # Live filter
```

```text
jq -c '.[]' array.json    → JSON array → NDJSON
jq -s '.' lines.ndjson    → NDJSON → JSON array
```

---

## PRACTICAL EXAMPLES

```bash
# Pretty-print API response:
curl -s https://api.example.com/users | jq '.'

# Get all names from user list:
curl -s https://api.example.com/users | jq '[.[] | .name]'

# Filter active users:
curl -s https://api.example.com/users | jq '[.[] | select(.active == true)]'

# Count items in array:
jq 'length' response.json

# Extract nested field from all items:
jq '.data[].attributes.email' response.json

# Find a specific item by id:
jq '.[] | select(.id == 42)' users.json

# Build minimal object from large payload:
jq '{id: .id, name: .name, email: .email}' user.json

# Strip all null fields:
jq 'del(.[] | nulls)' data.json

# Summarize prices:
jq '[.[] | .price] | {count: length, total: add, avg: (add/length)}' items.json

# Chain with grep/sed:
jq -r '.[] | .ip' servers.json | grep "^10."

# Health check as exit code (-e: false/null → exit 1):
curl -s https://api.example.com/health | jq -e '.status == "ok"' > /dev/null && echo OK || echo DOWN
```

---

## BLOCKCHAIN EXAMPLES

```bash
# Solana RPC response — extract lamports:
curl -s -X POST https://api.devnet.solana.com \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getBalance","params":["PUBKEY"]}' \
  | jq '.result.value'

# ETH JSON-RPC — extract hex block number (decode via printf — see curl sheet):
curl -s -X POST https://mainnet.infura.io/v3/KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  | jq -r '.result'

# Filter transactions by value:
jq '.transactions[] | select(.value > 0)' block.json
```

```text
⚠️ Wei balances and token IDs exceed 2^53 — jq ARITHMETIC on them
silently loses precision (floats). Extract them as strings with -r and
convert outside jq (python int(), printf for hex). See Gotchas.
```

---

## jq FLAGS REFERENCE

| Flag | Description |
|---|---|
| `-r` / `--raw-output` | No quotes on string output |
| `-R` / `--raw-input` | Treat input as raw string |
| `-s` / `--slurp` | Read all input into one array |
| `-c` / `--compact-output` | One-line output (NDJSON-friendly) |
| `-n` / `--null-input` | Use null as input (jq creates JSON) |
| `-e` / `--exit-status` | Exit 1 if output is false/null |
| `-j` / `--join-output` | No newline after each output |
| `-S` / `--sort-keys` | Sort object keys (stable diffs) |
| `-f file` | Read jq filter from file |
| `--arg name value` | Pass shell var into jq as string |
| `--argjson name val` | Pass shell var into jq as JSON |
| `--slurpfile name f.json` | Read a file into jq as $name (array) |

---

## DEBUGGING

```text
jq 'debug'                      → Print value to stderr, pass through
jq '. | debug | .name'          → Debug mid-pipeline
jq 'error("msg")'               → Raise error
jq 'try .x catch "failed"'      → Try/catch
jq 'try (.x | .y) catch .'      → Catch and return error message
```

---

## TIPS & GOTCHAS

- **Single-quote the filter, always** — double-quoting the whole filter lets the shell eat `$name` variables and backslashes before jq ever sees them. Shell values enter via `--arg`/`--argjson`, never string interpolation.
- **Regex backslashes are doubled** — jq regexes live inside jq strings, so `\w` must be written `"\\w"`. A single backslash is an invalid string escape and errors immediately.
- **Big numbers get mangled** — anything past 2^53 (wei, u64 token IDs, snowflake IDs) loses precision the moment jq does arithmetic on it. Keep them as strings end-to-end; convert in Python or with `printf` at the edges.
- **`select` streams; brackets collect** — `.[] | select(...)` emits bare objects (not valid JSON as a whole); wrap as `[.[] | select(...)]` or `map(select(...))` when a downstream tool expects one JSON array.
- **`.a.b` is safe on missing, not on mismatched** — missing keys yield null quietly, but indexing into a string/number errors. `?` (`.a.b?`) turns those errors into nulls; `..` + `?` is the standard deep-search combo.
- **`-e` turns jq into a test** — exit 1 on false/null makes jq the assertion in health checks and CI: `jq -e '.status == "ok"'`.
- **journal fields are all strings** — `journalctl -o json` gives `"PRIORITY": "3"`, not `3`; compare against `"3"` or pipe through `tonumber`.
- **Ubuntu 22.04 ships jq 1.6 (2018)** — jq 1.7 adds `pick`, `abs`, `ltrimstr` on non-strings, and saner number handling. apt won't give it to you; the static binary from the jq GitHub releases is a single-file drop-in.

---
*Last Updated: 2026-07*
