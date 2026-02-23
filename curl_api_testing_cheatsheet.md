# curl_api_testing_cheatsheet

# BASIC REQUESTS

---

curl [https://example.com](https://example.com/)                          → GET request
curl -s [https://example.com](https://example.com/)                       → Silent (no progress)
curl -i [https://example.com](https://example.com/)                       → Include response headers
curl -I [https://example.com](https://example.com/)                       → Headers ONLY (HEAD request)
curl -v [https://example.com](https://example.com/)                       → Verbose (req + res headers)
curl -vvv [https://example.com](https://example.com/)                     → Extra verbose (SSL details)
curl -o file.html [https://example.com](https://example.com/)             → Save output to file
curl -O [https://example.com/file.zip](https://example.com/file.zip)              → Save with remote filename
curl -L [https://example.com](https://example.com/)                       → Follow redirects
curl -w "\n%{http_code}\n" [https://example.com](https://example.com/)    → Show HTTP status code
curl --max-time 10 [https://example.com](https://example.com/)            → Timeout after 10s

---

## HTTP METHODS

curl -X GET    [https://api.example.com/users](https://api.example.com/users)
curl -X POST   [https://api.example.com/users](https://api.example.com/users)
curl -X PUT    [https://api.example.com/users/1](https://api.example.com/users/1)
curl -X PATCH  [https://api.example.com/users/1](https://api.example.com/users/1)
curl -X DELETE [https://api.example.com/users/1](https://api.example.com/users/1)

---

## HEADERS

curl -H "Authorization: Bearer TOKEN" [https://api.example.com](https://api.example.com/)
curl -H "Content-Type: application/json" [https://api.example.com](https://api.example.com/)
curl -H "Accept: application/json" [https://api.example.com](https://api.example.com/)
curl -H "X-API-Key: mykey" -H "X-Request-ID: abc123" [https://api.example.com](https://api.example.com/)

# Multiple headers

curl \
-H "Authorization: Bearer TOKEN" \
-H "Content-Type: application/json" \
-H "Accept: application/json" \
[https://api.example.com/users](https://api.example.com/users)

---

## SENDING DATA (POST / PUT / PATCH)

# JSON body

curl -X POST [https://api.example.com/users](https://api.example.com/users) \
-H "Content-Type: application/json" \
-d '{"name": "Alice", "email": "[alice@example.com](mailto:alice@example.com)"}'

# JSON from file

curl -X POST [https://api.example.com/users](https://api.example.com/users) \
-H "Content-Type: application/json" \
-d @payload.json

# Form data (application/x-www-form-urlencoded)

curl -X POST [https://api.example.com/login](https://api.example.com/login) \
-d "username=alice&password=secret"

# Multipart form (file upload)

curl -X POST [https://api.example.com/upload](https://api.example.com/upload) \
-F "file=@/path/to/file.pdf" \
-F "description=My file"

# Raw binary upload

curl -X PUT [https://api.example.com/file](https://api.example.com/file) \
-H "Content-Type: application/octet-stream" \
--data-binary @file.bin

---

## AUTHENTICATION

# Bearer token

curl -H "Authorization: Bearer eyJhbGciOi..." [https://api.example.com](https://api.example.com/)

# Basic auth

curl -u username:password [https://api.example.com](https://api.example.com/)
curl -H "Authorization: Basic $(echo -n 'user:pass' | base64)" [https://api.example.com](https://api.example.com/)

# API key in header

curl -H "X-API-Key: my_api_key" [https://api.example.com](https://api.example.com/)

# API key in query string

curl "[https://api.example.com/data?api_key=my_api_key](https://api.example.com/data?api_key=my_api_key)"

---

## QUERY PARAMETERS

curl "[https://api.example.com/users?page=1&limit=20](https://api.example.com/users?page=1&limit=20)"
curl -G [https://api.example.com/users](https://api.example.com/users) \
--data-urlencode "search=alice smith" \
--data-urlencode "page=1"

---

## RESPONSE HANDLING

# Pretty print JSON (pipe to jq)

curl -s [https://api.example.com/users](https://api.example.com/users) | jq '.'
curl -s [https://api.example.com/users](https://api.example.com/users) | jq '.[0]'
curl -s [https://api.example.com/users](https://api.example.com/users) | jq '.[].name'
curl -s [https://api.example.com/users](https://api.example.com/users) | jq '.[] | select(.active == true)'

# Save response to file

curl -s [https://api.example.com/data](https://api.example.com/data) -o response.json

# Capture status code separately

response=$(curl -s -o /tmp/body.json -w "%{http_code}" [https://api.example.com](https://api.example.com/))
echo "Status: $response"
cat /tmp/body.json | jq '.'

---

## SSL / TLS

curl -k [https://self-signed.example.com](https://self-signed.example.com/)           → Skip SSL verification (insecure)
curl --cacert ca.pem [https://example.com](https://example.com/)           → Use custom CA cert
curl --cert client.pem --key client.key [https://example.com](https://example.com/)  → Mutual TLS
curl --tlsv1.2 [https://example.com](https://example.com/)                → Force TLS version

---

## COOKIES

curl -c cookies.txt [https://example.com](https://example.com/)           → Save cookies to file
curl -b cookies.txt [https://example.com](https://example.com/)           → Send cookies from file
curl -b "session=abc123" [https://example.com](https://example.com/)      → Send cookie inline
curl -c cookies.txt -b cookies.txt [https://example.com](https://example.com/)  → Save + send

---

## TIMEOUTS & RETRY

curl --connect-timeout 5 [https://example.com](https://example.com/)      → Connection timeout (5s)
curl --max-time 30 [https://example.com](https://example.com/)            → Total timeout (30s)
curl --retry 3 [https://example.com](https://example.com/)                → Retry 3 times on fail
curl --retry 3 --retry-delay 2 [https://example.com](https://example.com/) → Retry with 2s delay
curl --retry 3 --retry-all-errors [https://example.com](https://example.com/) → Retry on any error

---

## PROXY

curl -x [http://proxy.example.com:8080](http://proxy.example.com:8080/) [https://api.example.com](https://api.example.com/)
curl -x socks5://127.0.0.1:1080 [https://api.example.com](https://api.example.com/)  → SOCKS5 proxy
curl --noproxy "localhost,127.0.0.1" [https://api.example.com](https://api.example.com/)

---

## USEFUL FLAGS REFERENCE

- s / --silent → No progress or error output
-S / --show-error → Show error even when -s is used
-i → Include response headers in output
-I → HEAD request (headers only)
-v → Verbose (shows full transaction)
-L → Follow redirects
-o file → Write output to file
-O → Write output to filename from URL
-d / --data → Request body data
-F / --form → Multipart form data
-H / --header → Add header
-u / --user → Username:password for basic auth
-X / --request → HTTP method
-b / --cookie → Send cookie
-c / --cookie-jar → Save cookies
-k / --insecure → Skip SSL verification
-w / --write-out → Custom output format after response
--max-time → Max total time
--connect-timeout → Connection timeout
--retry → Retry count
--compressed → Request compressed response (gzip)

---

## COMMON API TESTING PATTERNS

# Health check

curl -s -o /dev/null -w "%{http_code}" [https://api.example.com/health](https://api.example.com/health)

# Test REST CRUD

BASE="[https://api.example.com](https://api.example.com/)"
TOKEN="Bearer mytoken"

# Create

curl -X POST "$BASE/users" \
-H "Authorization: $TOKEN" \
-H "Content-Type: application/json" \
-d '{"name": "Alice"}'

# Read all

curl -s "$BASE/users" -H "Authorization: $TOKEN" | jq '.'

# Read one

curl -s "$BASE/users/1" -H "Authorization: $TOKEN" | jq '.'

# Update

curl -X PATCH "$BASE/users/1" \
-H "Authorization: $TOKEN" \
-H "Content-Type: application/json" \
-d '{"name": "Alice Updated"}'

# Delete

curl -X DELETE "$BASE/users/1" -H "Authorization: $TOKEN"

---

## BLOCKCHAIN / WEB3 API EXAMPLES

# Get ETH block number (JSON-RPC)

curl -X POST [https://mainnet.infura.io/v3/YOUR_KEY](https://mainnet.infura.io/v3/YOUR_KEY) \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'

# Get account balance

curl -X POST [https://mainnet.infura.io/v3/YOUR_KEY](https://mainnet.infura.io/v3/YOUR_KEY) \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0xAddress","latest"],"id":1}'

# Solana getBalance (via RPC)

curl -X POST [https://api.devnet.solana.com](https://api.devnet.solana.com/) \
-H "Content-Type: application/json" \
-d '{"jsonrpc":"2.0","id":1,"method":"getBalance","params":["YourPublicKeyHere"]}'

# QuickNode / Alchemy pattern

curl -X POST https://YOUR_ENDPOINT.quiknode.pro/TOKEN/ \
-H "Content-Type: application/json" \
-d '{"method":"eth_blockNumber","params":[],"id":1,"jsonrpc":"2.0"}'