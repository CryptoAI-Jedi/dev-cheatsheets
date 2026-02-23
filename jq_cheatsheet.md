# jq_cheatsheet

# WHAT IS jq?

---

jq is a lightweight, command-line JSON processor.
Think of it as sed/awk/grep — but for JSON.
Install: sudo apt install jq / sudo pacman -S jq / brew install jq

---

## BASIC USAGE

cat file.json | jq '.'              → Pretty-print JSON
curl -s [https://api.example.com](https://api.example.com/) | jq '.'
jq '.' file.json                    → Same as above (file directly)
jq -r '.' file.json                 → Raw output (no quotes on strings)
jq -c '.' file.json                 → Compact output (one line)
jq -n '{"key": "value"}'            → Create JSON from scratch (null input)
jq '.' <<< '{"name":"Alice"}'       → Inline JSON input

---

## BASIC FIELD ACCESS

# Given: {"name": "Alice", "age": 30}

jq '.name'                          → "Alice"
jq '.age'                           → 30
jq '.missing'                       → null  (no error)
jq '.missing // "default"'          → "default"  (alternative operator)
jq -r '.name'                       → Alice  (raw, no quotes)

---

## NESTED ACCESS

# Given: {"user": {"name": "Alice", "address": {"city": "KL"}}}

jq '.user.name'                     → "Alice"
jq '.user.address.city'             → "KL"
jq '.user | .name'                  → "Alice"  (pipe within jq)

---

## ARRAYS

# Given: [1, 2, 3, 4, 5]

jq '.[0]'                           → 1  (first element)
jq '.[-1]'                          → 5  (last element)
jq '.[2:4]'                         → [3, 4]  (slice)
jq '.[]'                            → Explode array (each element on own line)
jq 'length'                         → 5  (array length)
jq '.[0:3]'                         → [1, 2, 3]

# Array of objects

# Given: [{"name":"Alice","age":30},{"name":"Bob","age":25}]

jq '.[].name'                       → "Alice" \n "Bob"
jq '.[0].name'                      → "Alice"
jq 'length'                         → 2  (number of objects)

---

## TRANSFORMING OUTPUT

# Build new object

jq '{username: .name, years: .age}'

# Build array from fields

jq '[.name, .age]'

# From array of objects → new array of objects

jq '[.[] | {n: .name, a: .age}]'

# Extract specific fields from each item

jq '.[] | {name, age}'              → Shorthand (key = field name)

---

## FILTERING ARRAYS (select)

# Given: [{"name":"Alice","age":30},{"name":"Bob","age":25}]

jq '.[] | select(.age > 27)'        → Alice object only
jq '.[] | select(.name == "Bob")'   → Bob object only
jq '[.[] | select(.age >= 25)]'     → Array of matches
jq '.[] | select(.name | startswith("A"))'
jq '.[] | select(.role != null)'    → Has field
jq '.[] | select(.active == true)'

---

## STRING OPERATIONS

jq '.name | length'                 → String length
jq '.name | ascii_downcase'         → Lowercase
jq '.name | ascii_upcase'           → Uppercase
jq '.name | ltrimstr("prefix")'     → Remove prefix
jq '.name | rtrimstr("suffix")'     → Remove suffix
jq '.name | split(" ")'             → Split string → array
jq '.tags | join(", ")'             → Array → comma-separated string
jq '"\(.name) is \(.age) years old"' → String interpolation
jq '.name | test("^A")'             → Regex test → true/false
jq '.name | match("^(A\\w+)")'      → Regex match object
jq '.name | capture("(?P<first>\\w+)")'  → Named capture

---

## NUMBERS & MATH

jq '.price * .qty'                  → Multiply fields
jq '.price | floor'                 → Floor
jq '.price | ceil'                  → Ceiling
jq '.price | round'                 → Round
jq '.price | fabs'                  → Absolute value
jq '.price | sqrt'                  → Square root
jq '[.[] | .price] | add'          → Sum all prices
jq '[.[] | .price] | min'          → Min
jq '[.[] | .price] | max'          → Max
jq '[.[] | .price] | add / length' → Average

---

## ARRAY OPERATIONS

jq '[.[] | .price]'                 → Extract field into array (map)
jq 'map(.price)'                    → Same as above (cleaner)
jq 'map(. * 2)'                     → Double every element
jq 'map(select(. > 10))'           → Filter array elements
jq 'map(.name | ascii_upcase)'     → Transform each
jq 'sort'                           → Sort array of primitives
jq 'sort_by(.age)'                  → Sort objects by field
jq 'sort_by(.age) | reverse'       → Sort descending
jq 'unique'                         → Deduplicate array
jq 'unique_by(.name)'              → Deduplicate by field
jq 'group_by(.role)'               → Group objects by field
jq 'flatten'                        → Flatten nested arrays
jq 'flatten(1)'                     → Flatten one level
jq 'first'                          → First element
jq 'last'                           → Last element
jq 'nth(2)'                         → Third element (0-indexed)
jq 'reverse'                        → Reverse array
jq 'indices("x")'                  → Positions of "x"
jq 'contains([3])'                  → Check if array contains 3
jq 'inside([1,2,3])'               → Inverse of contains
jq 'any(. > 2)'                    → True if any element > 2
jq 'all(. > 0)'                    → True if all elements > 0
jq 'add'                            → Sum / concatenate array
jq 'del(.[2])'                      → Delete element at index 2

---

## OBJECT OPERATIONS

jq 'keys'                           → Array of object keys
jq 'values'                         → Array of object values
jq 'keys_unsorted'                  → Keys without sorting
jq 'to_entries'                     → [{key,value}] pairs
jq 'from_entries'                   → [{key,value}] → object
jq 'with_entries(.value += 1)'     → Transform each value
jq 'has("name")'                    → Check key exists
jq 'in({"a":1,"b":2})'             → Reverse has
jq '. + {"newkey": "val"}'         → Add/merge field
jq 'del(.unwanted)'                 → Remove field
jq '. | to_entries | map(select(.value != null)) | from_entries'  → Remove nulls

---

## CONDITIONALS

jq 'if .age > 18 then "adult" else "minor" end'
jq 'if .status == "active" then . else empty end'  → Filter with if
jq '.type // "unknown"'             → Default if null
jq 'if . == null then "N/A" elif . == 0 then "zero" else . end'

---

## VARIABLES & REDUCE

jq '. as $x | $[x.name](http://x.name/)'             → Bind to variable
jq '[.[] | . as $item | $[item.name](http://item.name/)]'

# reduce

jq 'reduce .[] as $x (0; . + $x)'  → Sum all numbers
jq 'reduce .[] as $x (0; . + $x.price)'  → Sum prices from objects

---

## COMBINING FILTERS

jq '.[] | .name, .age'             → Multiple outputs per item
jq '.[] | [.name, .age]'           → As array per item
jq '(.a + .b) / 2'                → Expression grouping

---

## PATHS

jq 'path(.user.name)'              → ["user","name"]
jq 'getpath(["user","name"])'      → Value at path
jq 'setpath(["user","name"]; "Bob")'  → Set value at path
jq 'delpaths([["user","age"]])'    → Delete paths

---

## TYPE FUNCTIONS

jq 'type'                           → "null","boolean","number","string","array","object"
jq 'numbers'                        → Filter: only numbers
jq 'strings'                        → Filter: only strings
jq 'booleans'                       → Filter: only booleans
jq 'arrays'                         → Filter: only arrays
jq 'objects'                        → Filter: only objects
jq 'nulls'                          → Filter: only nulls
jq 'scalars'                        → Filter: non-iterables
jq 'iterables'                      → Filter: arrays + objects
jq 'empty'                          → Produce no output (useful for filtering)

---

## MULTIPLE FILES & SLURP

jq -s '.'  file1.json file2.json   → Slurp multiple files into array
jq -s 'add' file1.json file2.json  → Merge JSON objects from two files
jq -s 'flatten' *.json             → Combine all JSON arrays

---

## INPUT / OUTPUT FORMATS

jq -r '.name'                       → Raw string output (no quotes)
jq -j '.name'                       → Raw output + no newline
jq -R '.'                           → Raw input (treat input as string)
jq -Rs '.'                          → Raw input + slurp into single string
jq -Rs 'split("\n")'               → Read lines into array
jq -r '.[]' file.json              → Print each element (no quotes)

---

## CSV / TSV OUTPUT

jq -r '.[] | [.name, .age, .city] | @csv'   → CSV format
jq -r '.[] | [.name, .age, .city] | @tsv'   → TSV format
jq -r '.[] | [.name, .age] | @csv' > out.csv

---

## PRACTICAL EXAMPLES

# Pretty-print API response

curl -s [https://api.example.com/users](https://api.example.com/users) | jq '.'

# Get all names from user list

curl -s [https://api.example.com/users](https://api.example.com/users) | jq '[.[] | .name]'

# Filter active users

curl -s [https://api.example.com/users](https://api.example.com/users) | jq '[.[] | select(.active == true)]'

# Count items in array

jq 'length' response.json

# Extract nested field from all items

jq '.data[].attributes.email' response.json

# Find a specific item by id

jq '.[] | select(.id == 42)' users.json

# Build minimal object from large payload

jq '{id: .id, name: .name, email: .email}' user.json

# Strip all null fields

jq 'del(.[] | nulls)' data.json

# Summarize prices

jq '[.[] | .price] | {count: length, total: add, avg: (add/length)}' items.json

# Chain with grep/sed

jq -r '.[] | .ip' servers.json | grep "^10\."

# Blockchain examples

## Solana RPC response — extract lamports

curl -s -X POST [https://api.devnet.solana.com](https://api.devnet.solana.com/) \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","id":1,"method":"getBalance","params":["PUBKEY"]}' \
| jq '.result.value'

## ETH JSON-RPC — decode hex block number

curl -s -X POST [https://mainnet.infura.io/v3/KEY](https://mainnet.infura.io/v3/KEY) \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
| jq -r '.result'

## Filter transactions by value

jq '.transactions[] | select(.value > 0)' block.json

---

## jq FLAGS REFERENCE

- r / --raw-output → No quotes on string output
-R / --raw-input → Treat input as raw string
-s / --slurp → Read all input into one array
-c / --compact-output → One-line output
-n / --null-input → Use null as input (jq creates JSON)
-e / --exit-status → Exit 1 if output is false/null
-j / --join-output → No newline after each output
-f file → Read jq filter from file
-arg name value → Pass shell var as string to jq
-argjson name val → Pass shell var as JSON to jq

---

## DEBUGGING

jq 'debug'                          → Print value to stderr, pass through
jq '. | debug | .name'             → Debug mid-pipeline
jq 'error("msg")'                  → Raise error
jq 'try .x catch "failed"'        → Try/catch
jq 'try (.x | .y) catch .'        → Catch and return error message