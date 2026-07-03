# curl API Testing Cheatsheet

> HTTP Swiss-army knife for testing REST, JSON-RPC, and LLM APIs from the terminal: request shaping, auth, response parsing with jq, and wire-level debugging.

---

## Table of Contents
- [Basic Requests](#basic-requests)
- [HTTP Methods](#http-methods)
- [Headers](#headers)
- [Sending Data (POST / PUT / PATCH)](#sending-data-post--put--patch)
- [Authentication](#authentication)
- [Query Parameters](#query-parameters)
- [Response Handling (jq)](#response-handling-jq)
- [SSL / TLS](#ssl--tls)
- [Cookies](#cookies)
- [Timeouts & Retry](#timeouts--retry)
- [Proxy](#proxy)
- [Debugging & Timing](#debugging--timing)
- [Common API Testing Patterns](#common-api-testing-patterns)
- [Blockchain / Web3 API Examples](#blockchain--web3-api-examples)
- [AI / LLM API Examples](#ai--llm-api-examples)
- [Useful Flags Reference](#useful-flags-reference)
- [Tips & Gotchas](#tips--gotchas)

---

## BASIC REQUESTS

```bash
curl https://example.com                    # GET request
curl -s https://example.com                 # Silent (no progress)
curl -i https://example.com                 # Include response headers
curl -I https://example.com                 # Headers ONLY (HEAD request)
curl -v https://example.com                 # Verbose (req + res headers)
curl -vvv https://example.com               # Extra verbose (SSL details)
curl -o file.html https://example.com       # Save output to file
curl -O https://example.com/file.zip        # Save with remote filename
curl -L https://example.com                 # Follow redirects
curl -w "\n%{http_code}\n" https://example.com   # Show HTTP status code
curl --max-time 10 https://example.com      # Timeout after 10s
curl -fsS https://example.com               # Script default: fail on 4xx/5xx, silent, but show errors
```

---

## HTTP METHODS

```bash
curl https://api.example.com/users              # GET (default — no -X needed)
curl -X POST https://api.example.com/users
curl -X PUT https://api.example.com/users/1
curl -X PATCH https://api.example.com/users/1
curl -X DELETE https://api.example.com/users/1
```

```text
💡 -d implies POST automatically — only use -X for PUT/PATCH/DELETE.
See Gotchas for why `-X GET` with a body misbehaves.
```

---

## HEADERS

```bash
curl -H "Authorization: Bearer TOKEN" https://api.example.com
curl -H "Content-Type: application/json" https://api.example.com
curl -H "Accept: application/json" https://api.example.com
curl -H "X-API-Key: mykey" -H "X-Request-ID: abc123" https://api.example.com
```

### Multiple headers

```bash
curl \
  -H "Authorization: Bearer TOKEN" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json" \
  https://api.example.com/users
```

---

## SENDING DATA (POST / PUT / PATCH)

### JSON body

```bash
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Alice", "email": "alice@example.com"}'
```

### JSON from file

```bash
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d @payload.json
```

### JSON with shell variables (heredoc — avoids quote hell)

```bash
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d @- <<EOF
{"name": "$NAME", "email": "$EMAIL"}
EOF
```

### Form data (application/x-www-form-urlencoded)

```bash
curl -X POST https://api.example.com/login \
  -d "username=alice&password=secret"
```

### Multipart form (file upload)

```bash
curl -X POST https://api.example.com/upload \
  -F "file=@/path/to/file.pdf" \
  -F "description=My file"
```

### Raw binary upload

```bash
curl -X PUT https://api.example.com/file \
  -H "Content-Type: application/octet-stream" \
  --data-binary @file.bin
```

---

## AUTHENTICATION

```bash
# Bearer token:
curl -H "Authorization: Bearer eyJhbGciOi..." https://api.example.com

# Basic auth:
curl -u username:password https://api.example.com
curl -H "Authorization: Basic $(echo -n 'user:pass' | base64)" https://api.example.com

# API key in header:
curl -H "X-API-Key: my_api_key" https://api.example.com

# API key in query string (⚠️ leaks into server logs + shell history — prefer headers):
curl "https://api.example.com/data?api_key=my_api_key"

# Credentials from ~/.netrc (keeps secrets out of history entirely):
curl -n https://api.example.com
```

---

## QUERY PARAMETERS

```bash
curl "https://api.example.com/users?page=1&limit=20"

# URL-encode values safely (spaces, special chars):
curl -G https://api.example.com/users \
  --data-urlencode "search=alice smith" \
  --data-urlencode "page=1"
```

---

## RESPONSE HANDLING (jq)

```bash
curl -s https://api.example.com/users | jq '.'                        # Pretty print
curl -s https://api.example.com/users | jq '.[0]'                     # First element
curl -s https://api.example.com/users | jq '.[].name'                 # Extract field
curl -s https://api.example.com/users | jq '.[] | select(.active == true)'   # Filter
```

```bash
# Save response to file:
curl -s https://api.example.com/data -o response.json

# Capture status code separately from body:
response=$(curl -s -o /tmp/body.json -w "%{http_code}" https://api.example.com)
echo "Status: $response"
cat /tmp/body.json | jq '.'
```

---

## SSL / TLS

```bash
curl -k https://self-signed.example.com          # Skip SSL verification (insecure)
curl --cacert ca.pem https://example.com         # Use custom CA cert
curl --cert client.pem --key client.key https://example.com   # Mutual TLS
curl --tlsv1.2 https://example.com               # Force TLS version
```

---

## COOKIES

```bash
curl -c cookies.txt https://example.com          # Save cookies to file
curl -b cookies.txt https://example.com          # Send cookies from file
curl -b "session=abc123" https://example.com     # Send cookie inline
curl -c cookies.txt -b cookies.txt https://example.com   # Save + send
```

---

## TIMEOUTS & RETRY

```bash
curl --connect-timeout 5 https://example.com     # Connection timeout (5s)
curl --max-time 30 https://example.com           # Total timeout (30s)
curl --retry 3 https://example.com               # Retry 3 times on transient fail
curl --retry 3 --retry-delay 2 https://example.com   # Retry with 2s delay
curl --retry 3 --retry-all-errors https://example.com   # Retry on ANY error (incl. 4xx/5xx)
```

---

## PROXY

```bash
curl -x http://proxy.example.com:8080 https://api.example.com
curl -x socks5://127.0.0.1:1080 https://api.example.com   # SOCKS5 (pairs with ssh -D)
curl --noproxy "localhost,127.0.0.1" https://api.example.com
```

---

## DEBUGGING & TIMING

```bash
# Where is the time going? (DNS vs connect vs TLS vs server):
curl -s -o /dev/null -w "dns: %{time_namelookup}s | connect: %{time_connect}s | tls: %{time_appconnect}s | ttfb: %{time_starttransfer}s | total: %{time_total}s\n" https://api.example.com

# Test a vhost against a SPECIFIC server (pre-DNS-cutover, or bypass CDN):
curl --resolve example.com:443:203.0.113.10 https://example.com

# Talk to a unix socket (Docker API without exposing a port):
curl --unix-socket /var/run/docker.sock http://localhost/containers/json | jq '.'

# Full wire trace to file:
curl --trace-ascii trace.txt https://api.example.com
```

---

## COMMON API TESTING PATTERNS

### Health check

```bash
curl -s -o /dev/null -w "%{http_code}" https://api.example.com/health
# Scriptable: -f makes curl exit non-zero on HTTP errors
curl -fsS https://api.example.com/health > /dev/null && echo OK || echo DOWN
```

### Test REST CRUD

```bash
BASE="https://api.example.com"
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
```

---

## BLOCKCHAIN / WEB3 API EXAMPLES

### Get ETH block number (JSON-RPC)

```bash
curl -X POST https://mainnet.infura.io/v3/YOUR_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```

### Get account balance

```bash
curl -X POST https://mainnet.infura.io/v3/YOUR_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_getBalance","params":["0xAddress","latest"],"id":1}'
```

### Convert hex JSON-RPC results to decimal

```bash
curl -s -X POST https://mainnet.infura.io/v3/YOUR_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}' \
  | jq -r '.result' | xargs printf "%d\n"
```

### Current gas price

```bash
curl -s -X POST https://mainnet.infura.io/v3/YOUR_KEY \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","method":"eth_gasPrice","params":[],"id":1}' \
  | jq -r '.result' | xargs printf "%d\n"     # Result in wei
```

### Solana getBalance (via RPC)

```bash
curl -X POST https://api.devnet.solana.com \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"getBalance","params":["YourPublicKeyHere"]}'
```

### QuickNode / Alchemy pattern

```bash
curl -X POST https://YOUR_ENDPOINT.quiknode.pro/TOKEN/ \
  -H "Content-Type: application/json" \
  -d '{"method":"eth_blockNumber","params":[],"id":1,"jsonrpc":"2.0"}'
```

### Spot prices (CoinGecko — public, no key)

```bash
curl -s "https://api.coingecko.com/api/v3/simple/price?ids=bitcoin,ethereum&vs_currencies=usd" | jq '.'
```

---

## AI / LLM API EXAMPLES

### Anthropic Messages API

```bash
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-sonnet-4-6",
    "max_tokens": 1024,
    "messages": [{"role": "user", "content": "Hello"}]
  }' | jq -r '.content[0].text'
```

### OpenAI Chat Completions

```bash
curl https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [{"role": "user", "content": "Hello"}]
  }' | jq -r '.choices[0].message.content'
```

### OpenAI-compatible providers (OpenRouter, Together, vLLM, etc.)

Same payload schema as OpenAI — only the base URL and key change:

```bash
curl https://openrouter.ai/api/v1/chat/completions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "MODEL_ID", "messages": [{"role": "user", "content": "Hello"}]}'
```

### Streaming (SSE) — the -N flag is the trick

```bash
curl -N https://api.openai.com/v1/chat/completions \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-4o-mini", "stream": true, "messages": [{"role": "user", "content": "Hello"}]}'
# Without -N curl buffers output — the stream looks frozen, then dumps at once
```

### Local models (Ollama)

```bash
curl http://localhost:11434/api/tags | jq '.models[].name'   # List installed models

curl http://localhost:11434/api/chat -d '{
  "model": "llama3.2",
  "stream": false,
  "messages": [{"role": "user", "content": "Hello"}]
}' | jq -r '.message.content'
```

---

## USEFUL FLAGS REFERENCE

| Flag | Description |
|---|---|
| `-s` / `--silent` | No progress or error output |
| `-S` / `--show-error` | Show error even when -s is used |
| `-f` / `--fail` | Exit non-zero on HTTP 4xx/5xx (scripts/CI) |
| `-i` | Include response headers in output |
| `-I` | HEAD request (headers only) |
| `-v` | Verbose (shows full transaction) |
| `-L` | Follow redirects |
| `-o file` | Write output to file |
| `-O` | Write output to filename from URL |
| `-d` / `--data` | Request body data (implies POST) |
| `--data-urlencode` | Body data, URL-encoded |
| `-G` | Send -d data as query string (GET) |
| `-F` / `--form` | Multipart form data |
| `-H` / `--header` | Add header |
| `-u` / `--user` | Username:password for basic auth |
| `-n` / `--netrc` | Read credentials from ~/.netrc |
| `-X` / `--request` | HTTP method override |
| `-b` / `--cookie` | Send cookie |
| `-c` / `--cookie-jar` | Save cookies |
| `-k` / `--insecure` | Skip SSL verification |
| `-w` / `--write-out` | Custom output format after response |
| `-N` / `--no-buffer` | Disable output buffering (SSE/streaming) |
| `--max-time` | Max total time |
| `--connect-timeout` | Connection timeout |
| `--retry` | Retry count (transient errors) |
| `--retry-all-errors` | Retry on any error incl. HTTP codes |
| `--resolve` | Pin a hostname to a specific IP |
| `--unix-socket` | Send request over a unix socket |
| `--compressed` | Request compressed response (gzip) |

---

## TIPS & GOTCHAS

- **Quote any URL containing `?` or `&`** — unquoted, the shell treats `&` as "run in background" and silently truncates your query string.
- **Let curl pick the method** — `-d` implies POST; `-I` implies HEAD. `-X GET` with `-d` sends a GET-with-body most servers ignore, and `-X` breaks redirect method handling with `-L`.
- **Single-quote JSON bodies** — protects the inner double quotes from the shell. Need variables inside? Use the heredoc pattern (`-d @- <<EOF`) instead of quote gymnastics.
- **`-s` eats error messages** — always pair as `-sS` (or `-fsS` in scripts) so failures are silent-but-visible.
- **`--retry` alone won't retry HTTP 500s** — it only covers transient/connection errors; add `--retry-all-errors` for status-code retries.
- **Secrets hygiene** — keys in query strings land in server logs; keys in commands land in shell history. Use env vars, `~/.netrc`, or a leading space with `HISTCONTROL=ignorespace`.
- **JSON-RPC returns hex** — `0x...` block numbers and wei balances; pipe through `xargs printf "%d\n"` for decimals (see Web3 section).
- **curl ≥7.82 has `--json '...'`** — shorthand that sets both Content-Type and Accept. Ubuntu 22.04 ships 7.81, so it's absent there; the `-H` + `-d` form works everywhere.

---
*Last Updated: 2026-07*
