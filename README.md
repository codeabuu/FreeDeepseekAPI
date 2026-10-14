# FreeDeepseekAPI

<p align="center">
  <strong>Local OpenAI-compatible API proxy for DeepSeek Web Chat</strong>
</p>

<p align="center">
  <a href="https://github.com/ForgetMeAI/FreeDeepseekAPI/blob/main/LICENSE"><img alt="License MIT" src="https://img.shields.io/badge/license-MIT-green.svg" /></a>
  <img alt="Node.js 18 plus" src="https://img.shields.io/badge/node-18%2B-339933.svg" />
  <img alt="No npm dependencies" src="https://img.shields.io/badge/dependencies-0-blue.svg" />
  <img alt="OpenAI compatible" src="https://img.shields.io/badge/OpenAI-compatible-111111.svg" />
</p>

<p align="center">
  <a href="#-quick-start">Quick start</a> •
  <a href="#-features">Features</a> •
  <a href="#-request-examples">Examples</a> •
  <a href="#-models">Models</a> •
  <a href="#-endpoints">Endpoints</a> •
  <a href="#-open-webui">Open WebUI</a>
</p>

FreeDeepseekAPI spins up a local API server for **DeepSeek Web Chat** (`chat.deepseek.com`) and lets you connect DeepSeek Web to Open WebUI, LiteLLM, Hermes, Claude Code, OpenAI SDK-style clients, and other OpenAI-compatible tools.

The project works through your regular logged-in DeepSeek account in a separate Chrome profile. The local server accepts API requests and then talks to DeepSeek Web itself using the saved browser session.

> ⚠️ This is an experimental web-chat proxy. DeepSeek may change its internal Web API without warning. For production use cases, the official paid DeepSeek API is more reliable.

ForgetMeAI: https://t.me/forgetmeai

---

## Navigation

- [What this gives you](#-what-this-gives-you)
- [Features](#-features)
- [Quick start](#-quick-start)
- [Running on Windows](#-running-on-windows)
- [Running on Linux / Chromium](#-running-on-linux--chromium)
- [Running on VPS / headless](#-running-on-vps--headless)
- [Persistent service on VPS (systemd)](#-persistent-service-on-vps-systemd)
- [Rootless Podman](#-rootless-podman)
- [Diagnostics / doctor](#-diagnostics--doctor)
- [Session reuse and chat reset](#-session-reuse-and-chat-reset)
- [Multi-account pool](#-multi-account-pool)
- [Console auth ideas](#-console-auth-ideas)
- [Verify it works](#-verify-it-works)
- [Request examples](#-request-examples)
  - [Chat Completions](#chat-completions)
  - [Reasoning](#reasoning)
  - [Web search](#web-search)
  - [Streaming](#streaming)
  - [Anthropic Messages API](#anthropic-messages-api)
  - [OpenAI Responses API](#openai-responses-api)
  - [Tool calling](#tool-calling)
- [Models](#-models)
- [Endpoints](#-endpoints)
- [Open WebUI](#-open-webui)
- [Refresh the login](#-refresh-the-login)
- [Project status](#-project-status)

---

## ✨ What this gives you

- Use DeepSeek Web as a local API endpoint.
- Connect DeepSeek to Open WebUI and other OpenAI-compatible clients.
- Get plain JSON responses or streaming SSE.
- Use reasoning models with a separate `reasoning_content`.
- Work with the Anthropic Messages API shim for Claude Code / the Anthropic SDK.
- Use the OpenAI Responses API shim for newer OpenAI/Codex-style clients.
- Keep separate web sessions for different agents/users.

## 🚀 Features

- **OpenAI-compatible API:** `POST /v1/chat/completions`
- **Anthropic-compatible shim:** `POST /v1/messages`
- **OpenAI Responses shim:** `POST /v1/responses`
- **Streaming:** SSE chunks and plain non-stream JSON responses
- **Reasoning output:** a separate `reasoning_content` for thinking models
- **Tool calling:** parsing OpenAI tools, Anthropic tools, and Responses function tools
- **Model capabilities:** `GET /v1/model-capabilities` with alias → real web mode
- **Agent sessions:** a separate DeepSeek session per `user` / agent id
- **Session recovery:** automatic reset of stale chains/sessions
- **Zero dependencies:** Node.js 18+ for the proxy (`npm run auth` needs Node.js 22+), no npm dependencies

---

## ⚡ Quick start

```bash
git clone https://github.com/ForgetMeAI/FreeDeepseekAPI.git
cd FreeDeepseekAPI
npm run auth
npm start
```

`npm run auth` opens the auth menu:

1. choose option `1`;
2. log in to DeepSeek in the separate Chrome profile;
3. send a short message such as `ok`;
4. return to the terminal and press Enter.

> `npm run auth` uses the WebSocket built into Node.js, so it needs
> Node.js 22+. The proxy itself runs on Node.js 18+; on older Node, import
> an existing `deepseek-auth.json` with `npm run auth:import`.

`npm start` shows the startup menu:

- `1` — log in / refresh the DeepSeek login
- `2` — import an auth file / cookies
- `3` — show models and statuses
- `4` — start the proxy (default, Enter)
- `5` — exit

You can put settings in `.env` (template: `.env.example`): `npm start`,
`npm run auth`, and `npm run doctor` read it automatically. Environment
variables take precedence over values from `.env`.

For headless/CI runs without the menu:

```bash
NON_INTERACTIVE=1 npm start
# or
SKIP_ACCOUNT_MENU=1 npm start
```

By default the server listens on:

```text
http://localhost:9655
```

By default the proxy is reachable only from this computer. To allow access from
the network, explicitly set the address and a separate proxy key:

```bash
HOST=0.0.0.0 PROXY_API_KEY='replace-with-a-long-random-value' npm start
```

After that, pass the key as `Authorization: Bearer <key>` or
`x-api-key: <key>` (that's how the Anthropic SDK sends it). Without
`PROXY_API_KEY`, non-health endpoints stay unauthenticated, so don't
expose such an instance to the network.

Browser requests are allowed from loopback origins. If the UI is served from
a different address, add its exact origin as a comma-separated value, for
example `PROXY_CORS_ORIGINS=https://ui.example.com,http://192.168.1.20:3000`.

---

## 🪟 Running on Windows

```powershell
git clone https://github.com/ForgetMeAI/FreeDeepseekAPI.git
cd FreeDeepseekAPI
npm run auth
npm start
```

If Chrome is installed in a non-standard location, specify the path explicitly:

```powershell
$env:CHROME_PATH="C:\Program Files\Google\Chrome\Application\chrome.exe"
npm run auth
```

If Chrome isn't found, `npm run auth` now prints ready-made instructions for Windows/macOS/Linux instead of a cryptic stack trace.

---

## 🐧 Running on Linux / Chromium

```bash
git clone https://github.com/ForgetMeAI/FreeDeepseekAPI.git
cd FreeDeepseekAPI
CHROME_PATH=$(which chromium) npm run auth
npm start
```

If your Chromium binary has a different name:

```bash
CHROME_PATH=$(which chromium-browser) npm run auth
# or
CHROME_PATH=$(which google-chrome) npm run auth
```

---

## 🖥 Running on VPS / headless

The most reliable flow without Chrome on the server:

1. On your home PC, where a GUI/Chrome is available:

```bash
npm run auth
```

2. Copy `deepseek-auth.json` to the VPS:

```bash
scp deepseek-auth.json user@your-vps:/opt/FreeDeepseekAPI/deepseek-auth.json
```

3. On the VPS, import/verify the file and set safe permissions:

```bash
cd /opt/FreeDeepseekAPI
npm run auth:import -- --input ./deepseek-auth.json
npm run doctor -- --offline
```

4. Start the proxy without the interactive menu:

```bash
NON_INTERACTIVE=1 npm start
```

You can import not only a ready-made `deepseek-auth.json`, but also a browser cookie export:

```bash
DEEPSEEK_TOKEN="<token>" npm run auth:import -- --input ./cookies.json
```

> Important: `deepseek-auth.json` grants access to your DeepSeek Web login. Don't commit it, don't publish it, and keep it with `0600` permissions.

---

## 🛠 Persistent service on VPS (systemd)

The no-container option: the proxy runs as a system service under a dedicated
user, restarts on failures, and starts together with the server. Requires
Node.js 18+ (`node` in `/usr/bin` or `/usr/local/bin`).

```bash
# 1. User and code
sudo useradd --system --home /opt/FreeDeepseekAPI --shell /usr/sbin/nologin freedeepseek
sudo git clone https://github.com/ForgetMeAI/FreeDeepseekAPI /opt/FreeDeepseekAPI

# 2. Secrets: the auth file from `npm run auth` on your home PC, and a key for clients
sudo install -d -m 0750 -o root -g freedeepseek /etc/free-deepseek-api
sudo install -m 0600 -o freedeepseek -g freedeepseek ./deepseek-auth.json /etc/free-deepseek-api/deepseek-auth.json
openssl rand -hex 32 | sudo install -m 0600 -o freedeepseek -g freedeepseek /dev/stdin /etc/free-deepseek-api/proxy-api-key

# 3. Service
sudo cp /opt/FreeDeepseekAPI/deploy/free-deepseek-api.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable --now free-deepseek-api

# 4. Verification
curl -fsS http://127.0.0.1:9655/readyz
sudo -u freedeepseek env DEEPSEEK_AUTH_PATH=/etc/free-deepseek-api/deepseek-auth.json node /opt/FreeDeepseekAPI/scripts/doctor.js
journalctl -u free-deepseek-api -f
```

- The service listens only on `127.0.0.1:9655` and requires a key
  (`Authorization: Bearer <key>` or `x-api-key`). The key lives in
  `/etc/free-deepseek-api/proxy-api-key`.
- External access goes through a reverse proxy with HTTPS (nginx/Caddy) or an
  SSH tunnel. If you need to listen on the network directly, set `HOST=0.0.0.0`
  in `/etc/free-deepseek-api/env` (put any `DEEPSEEK_*` settings there too).
- Updating: `cd /opt/FreeDeepseekAPI && sudo git pull && sudo systemctl restart free-deepseek-api`.
- A new DeepSeek login: install the file with the same command so the owner
  and permissions are preserved, then restart the service:
  `sudo install -m 0600 -o freedeepseek -g freedeepseek ./deepseek-auth.json /etc/free-deepseek-api/deepseek-auth.json && sudo systemctl restart free-deepseek-api`.

---

## 🦭 Rootless Podman

The container is meant only for non-interactive proxy runs. Do the browser
authorization on the host with `npm run auth`: the auth scripts and
`deepseek-auth.json` are not copied into the image.

Run Podman as a regular user, without `sudo`.

1. Build the local image:

```bash
podman build --format docker --tag localhost/free-deepseek-api:local --file Containerfile .
```

2. Pass the DeepSeek auth and a separate proxy key through Podman secrets:

```bash
podman secret create --replace free-deepseek-auth ./deepseek-auth.json

printf 'Proxy API key: '
IFS= read -r -s PROXY_API_KEY
printf '\n'
printf '%s' "$PROXY_API_KEY" |
  podman secret create --replace free-deepseek-proxy-key -
```

Use a long random key. The value stays in the current shell's `PROXY_API_KEY`
variable so you can test the API; it never lands in the image or on the
Podman command line.

3. Start the container with minimal privileges:

```bash
podman run --detach \
  --name free-deepseek-api \
  --publish 127.0.0.1:9655:9655 \
  --secret free-deepseek-auth,target=deepseek-auth.json,uid=1000,gid=1000,mode=0400 \
  --secret free-deepseek-proxy-key,target=proxy-api-key,uid=1000,gid=1000,mode=0400 \
  --read-only \
  --cap-drop=ALL \
  --security-opt=no-new-privileges \
  localhost/free-deepseek-api:local
```

Inside the container, `NON_INTERACTIVE=1`, `HOST=0.0.0.0`, and the paths to
both secrets are preset. `REQUIRE_PROXY_API_KEY=1` prevents the container from
starting if the key secret is missing or empty. On the host the port is
published only on `127.0.0.1`; don't remove that address without a separate
network firewall/access policy.

4. Check liveness, account readiness, and the protected endpoint:

```bash
podman healthcheck run free-deepseek-api
curl --fail http://127.0.0.1:9655/readyz
curl --fail \
  -H "Authorization: Bearer $PROXY_API_KEY" \
  http://127.0.0.1:9655/v1/models
```

The built-in healthcheck probes the local `/health` (is the process alive).
`--format docker` is required at build time: in OCI format (Podman's default)
the `HEALTHCHECK` instruction is silently dropped.
`/readyz` additionally returns `503` if no DeepSeek auth account is currently
ready to serve requests. Container diagnostics:

```bash
podman logs free-deepseek-api
podman inspect --format '{{.State.Health.Status}}' free-deepseek-api
```

Stop and remove the container along with the saved Podman secrets:

```bash
podman stop free-deepseek-api
podman rm free-deepseek-api
podman secret rm free-deepseek-auth free-deepseek-proxy-key
unset PROXY_API_KEY
```

When rotating the auth or proxy key, replace the corresponding secret and
recreate the container, so the behavior doesn't depend on the Podman version.

---

## 🩺 Diagnostics / doctor

```bash
npm run doctor
# without network requests to DeepSeek:
npm run doctor -- --offline
```

`doctor` checks:

- whether `deepseek-auth.json` / `DEEPSEEK_AUTH_DIR` is found;
- whether the JSON is valid;
- whether `token`, `cookie`, `wasmUrl` are present;
- whether the file permissions are safe on macOS/Linux (`0600`);
- on a normal run, whether the DeepSeek PoW endpoint is reachable.

If you see `data.biz_data is null`, `fetch failed`, `401/403/429`, or Hermes/OpenCode can't see the models — run `npm run doctor` first.

---

## ♻️ Session reuse and chat reset

FreeDeepseekAPI doesn't create a new DeepSeek chat for every HTTP request without a reason. Here's the logic:

- one `x-agent-session`, `session`, or `user` → one DeepSeek chat session;
- if the session id already exists, the proxy reuses it and continues the chain via `parent_message_id`;
- agents (Hermes, Cline, Claude Code, OpenCode…) send the whole history with every request. The proxy remembers which messages are already in the DeepSeek chat and sends **only the new ones** to the live chat (tool results, the new question) plus a short reminder about the tool call format. Previously the entire dialog (up to 80,000 characters) was resent at every step, the chat context grew quadratically, and after a dozen steps DeepSeek started answering with nothing, wrote "Content is too long", or stopped calling tools (#23, #30);
- if the client changed the history (compressed/edited it), or changed the system prompt or tool set, a new DeepSeek chat opens with the full prompt;
- auto-reset happens on TTL, a DeepSeek session error, an overly long message chain, or when the chat has accumulated more than `DEEPSEEK_MAX_SESSION_CHARS` characters (by default 3 × `DEEPSEEK_MAX_PROMPT_CHARS`);
- parallel requests from the same agent (for example, background reviews in Hermes) don't get mixed into one chat: the second request is served in a separate temporary DeepSeek chat;
- the local history is stored as a short context so a new DeepSeek session can continue the conversation.
- long agent requests are trimmed to `DEEPSEEK_MAX_PROMPT_CHARS` before sending (by default 80,000 characters): the start of the task, recent tool results, and the tool adapter are kept;
- if the client already sent multi-turn history, the local recovery history isn't appended a second time;
- an empty response is retried at most `DEEPSEEK_MAX_RETRIES` times (by default 2), and the context shrinks on every retry.
- long generations (thinking) aren't cut off by a timeout: the stream is bounded by the gap without data (`DEEPSEEK_STREAM_IDLE_TIMEOUT_MS`, 60 s by default) and the overall limit `DEEPSEEK_STREAM_MAX_MS` (10 minutes).

Set the agent/session explicitly:

```bash
curl -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "x-agent-session: my-agent" \
  -d '{"model":"deepseek-chat","messages":[{"role":"user","content":"Hello"}]}'
```

List active sessions:

```bash
curl http://localhost:9655/v1/sessions
```

Reset one session:

```bash
curl -X POST "http://localhost:9655/reset-session?agent=my-agent"
```

Reset all sessions:

```bash
curl -X POST "http://localhost:9655/reset-session?agent=all"
```

Why chats still show up in DeepSeek Web: the proxy works through the internal Web Chat API, and DeepSeek keeps the real chat sessions on its side. That's normal for a web proxy. The point of session reuse is to avoid spawning new chats unnecessarily and to reset cleanly only when the chain has expired or broken.

---

## 👥 Multi-account pool

You can connect several auth files. The right model is a sticky account per agent/session — the proxy doesn't switch accounts inside a live DeepSeek session. If an account gets `401/403/429` and goes into cooldown, the session is safely reset and the next request can move to another available account.

Option 1 — a directory of auth files:

```bash
mkdir -p accounts
cp deepseek-auth-main.json accounts/main.json
cp deepseek-auth-backup.json accounts/backup.json
chmod 600 accounts/*.json
DEEPSEEK_AUTH_DIR=./accounts NON_INTERACTIVE=1 npm start
```

Option 2 — a list of files:

```bash
DEEPSEEK_AUTH_PATH="./accounts/main.json,./accounts/backup.json" NON_INTERACTIVE=1 npm start
```

How the pool works:

- a new agent/session gets an available account round-robin;
- the chosen account is pinned to the session (`sticky`);
- on `401`, `403`, `429` the account goes into cooldown and the request is retried on another ready account;
- a DeepSeek "Too many messages" response also counts as a rate limit: the account goes into a short cooldown (`DEEPSEEK_RATE_LIMIT_COOLDOWN_MS`, 60 s by default); if there are no other accounts, the client gets a `429` with `Retry-After`;
- if a session's sticky account goes into cooldown, the old DeepSeek session is reset so we don't keep hammering a rate-limited/expired account;
- account status is visible in `/health` without auth file paths or file names;
- auth files must be stored with `0600` permissions.

Configure the cooldown:

```bash
DEEPSEEK_ACCOUNT_COOLDOWN_MS=600000 npm start
```

---

## 🔑 Console auth ideas

The password flow from PR #3 is doable, but it's safer not to store the password and not to make it the default. A proper implementation:

1. `npm run auth:console` asks for email/phone and password via a hidden prompt.
2. The password stays only in process memory; it's never written to files/logs/history.
3. The script replays the Web login flow via `fetch`/CDP: it gets the captcha/verify challenge, hands the person a link/code, and waits for confirmation.
4. After a successful login, only a standard-format `deepseek-auth.json` is saved.
5. If DeepSeek asks for a captcha/2FA, the script honestly says "open the link, pass the check, press Enter" instead of trying to bypass the protection.
6. For a VPS, the mode `auth:console --no-save-password --output deepseek-auth.json` is preferable.

Minimal safe MVP: console auth is interactive only, with no env password. An acceptable automation option: `DEEPSEEK_EMAIL=... npm run auth:console`, but the password is still entered via a hidden prompt.

---

## ✅ Verify it works

```bash
curl http://localhost:9655/
curl http://localhost:9655/v1/models
curl http://localhost:9655/v1/model-capabilities
```

If everything is fine, `/health` returns the server status, the list of supported aliases, and `config_ready: true`.

---

## 🧪 Request examples

### Chat Completions

```bash
curl -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "messages": [{"role": "user", "content": "Hi! Answer in one sentence."}],
    "stream": false
  }'
```

### Reasoning

```bash
curl -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-reasoner",
    "messages": [{"role": "user", "content": "Answer briefly: why is the sky blue?"}],
    "stream": false
  }'
```

With `stream: true`, an ordinary response (a request without `tools`) is streamed
to the client as DeepSeek writes it: first the reasoning, then the text. If the
request has `tools`, the response may turn out to be a tool call, so the proxy
waits for it to finish and delivers it whole, and while waiting it sends an SSE
comment `: keep-alive` every `DEEPSEEK_STREAM_KEEPALIVE_MS` (10 s) so the client
doesn't drop the connection on a timeout. An error that happens after the stream
has already started arrives as an `error` event inside the stream (the OpenAI and
Anthropic SDKs turn it into an exception); before the stream starts, it arrives
as a normal HTTP status.

For reasoning models, the API returns the chain of thought separately from the final answer:

- non-stream: `choices[0].message.reasoning_content`
- stream: `choices[0].delta.reasoning_content`
- usage: `usage.completion_tokens_details.reasoning_tokens`

`reasoning_tokens` is a rough estimate from the `THINK` text extracted from DeepSeek Web, because the web stream doesn't report official token usage for reasoning separately.

### Web search

```bash
curl -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat-search",
    "messages": [{"role": "user", "content": "Find a recent fact about DeepSeek and answer briefly."}],
    "stream": false
  }'
```

### Streaming

```bash
curl -N -X POST http://localhost:9655/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "messages": [{"role": "user", "content": "Write a short joke."}],
    "stream": true
  }'
```

### Anthropic Messages API

```bash
curl -X POST http://localhost:9655/v1/messages \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "max_tokens": 512,
    "messages": [{"role": "user", "content": "Reply with exactly OK"}],
    "stream": false
  }'
```

For Claude Code you can point it at the backend directly:

```bash
export ANTHROPIC_BASE_URL="http://127.0.0.1:9655"
export ANTHROPIC_AUTH_TOKEN="dummy-key"
export CLAUDE_CODE_ENABLE_GATEWAY_MODEL_DISCOVERY=1
claude --model deepseek-chat
```

### OpenAI Responses API

```bash
curl -X POST http://localhost:9655/v1/responses \
  -H "Content-Type: application/json" \
  -d '{
    "model": "deepseek-chat",
    "input": "Reply with exactly OK",
    "stream": false
  }'
```

### Tool calling

FreeDeepseekAPI accepts:

- OpenAI `tools`;
- Anthropic `tools`;
- Responses API function tools.

The proxy asks DeepSeek to return a strict JSON tool call, but it can also parse fallback formats:

- `TOOL_CALL:`
- fenced JSON with an explicit `tool_call`, `tool_calls`, or `function_call` envelope
- `<tool_call>...</tool_call>`
- DeepSeek DSML (`<｜DSML｜tool_calls>...`) and the Web variant with `<｜｜DSML｜｜ Tool Calls>`

Typical model mistakes are fixed during parsing: unescaped backslashes in Windows
paths (`"C:\git\src\index.ts"`) and real newlines inside string arguments. Code
blocks in another language (```` ```csharp ````, ```` ```python ````) are never
treated as a tool call.

---

## 🧠 Models

`GET /v1/models` returns only the aliases that are currently verified and working through this proxy.

### Unified DeepSeek Web mode (since 2026-09-10)

On September 10, 2026, DeepSeek merged the "Fast", "Expert", and "Vision" modes
in the web chat into a single smart mode on the **DeepSeek-V4.1-Flash** model,
and on September 14 it disabled V4 Pro. There's no model switcher anymore; only
"Deep thinking" (`thinking_enabled`) and "Search" (`search_enabled`) remain (#31).

So all aliases now send the same `model_type: "default"` to the Web API and
differ only in those two flags. The old names (`deepseek-expert`,
`deepseek-v4-pro`, …) are kept so client configs don't break, and are marked
`deprecated` in `/v1/model-capabilities`. If DeepSeek changes the `model_type`
value again, you can override it with `DEEPSEEK_MODEL_TYPE` (an empty value
means don't send the field).

### Working aliases

| Alias | Deep thinking | Web search | Notes |
| --- | --- | --- | --- |
| `deepseek-chat` | no | no | basic chat |
| `deepseek-default` | no | no | compatible alias |
| `deepseek-v4-flash` | no | no | official API name |
| `deepseek-v4.1-flash` | no | no | current web chat model |
| `deepseek-v3` | no | no | deprecated alias |
| `deepseek-reasoner` | yes | no | `thinking_enabled=true` |
| `deepseek-r1` | yes | no | R1-compatible alias |
| `deepseek-chat-search` | no | yes | web search |
| `deepseek-default-search` | no | yes | web search alias |
| `deepseek-reasoner-search` | yes | yes | reasoning + search |
| `deepseek-r1-search` | yes | yes | R1-compatible + search |
| `deepseek-expert` | yes | no | deprecated: "Expert" merged into the unified mode |
| `deepseek-v4-pro` | yes | no | deprecated: V4 Pro disabled, V4.1-Flash responds |
| `deepseek-expert-search` | yes | yes | deprecated alias, now working |

### Images (verified against the live DeepSeek Web API)

On the live DeepSeek Web API, on October 7, 2026, PNG/JPEG transfer through
`deepseek-vision` was verified: multipart upload, file processing from `PENDING`
through `PARSING` to `SUCCESS`, a file ID reference in the completion, and a
visual answer with readable text, colors, and layout. The model called nearly
square rectangles squares, so shape-recognition accuracy needs to be checked for
a specific task. Image uploads
are enabled by default. To disallow sending images to DeepSeek, explicitly
set `DEEPSEEK_IMAGE_UPLOAD=0`:

```bash
DEEPSEEK_IMAGE_UPLOAD=0 npm start
```

- inline images are accepted: OpenAI `image_url` / Responses
  `input_image` with `data:image/...;base64,...`, and Anthropic `image` with
  `source.type: "base64"` (including inside `tool_result`);
- `https://...` links are not downloaded (SSRF protection) — the model only
  sees the address;
- PNG, JPEG, WebP, and GIF are supported; other formats (such as SVG)
  remain a text marker;
- in an already-open chat, only images from new messages are uploaded; in a new
  chat, the last `DEEPSEEK_MAX_IMAGES` (4) images of the conversation are
  uploaded; a single image can be up to `DEEPSEEK_MAX_IMAGE_BYTES` (7 MB), and
  the whole request up to 10 MB; already-uploaded images are not uploaded again
  (30-minute cache);
- if DeepSeek fails to process an image, the request fails with a `502
  image_upload_failed` error rather than a quiet "blind" answer;
- with uploads enabled, `deepseek-vision` appears in `/v1/models` (it's the
  same unified mode).

With `DEEPSEEK_IMAGE_UPLOAD=0`, images are replaced with the short marker `[Image
attached; not visible through this proxy]`; their bytes are never uploaded to DeepSeek.

Full mapping:

```bash
curl http://localhost:9655/v1/model-capabilities
```

---

## 🔌 Endpoints

| Method | Path | Purpose |
| --- | --- | --- |
| `GET` | `/` or `/health` | proxy status |
| `GET` | `/v1/models` | list of working OpenAI-compatible aliases |
| `GET` | `/v1/model-capabilities` | full mapping of aliases, real model, capabilities |
| `POST` | `/v1/chat/completions` | OpenAI-compatible Chat Completions |
| `POST` | `/v1/messages` | Anthropic Messages API shim |
| `POST` | `/v1/responses` | OpenAI Responses API shim |
| `GET` | `/v1/sessions` | active local agent sessions |
| `POST` | `/reset-session?agent=<id>` | reset one session |
| `POST` | `/reset-session?agent=all` | reset all sessions |

---

## 🖥 Open WebUI

Base URL for Open WebUI in Docker:

```text
http://host.docker.internal:9655/v1
```

For a local run without Docker:

```text
http://localhost:9655/v1
```

If `PROXY_API_KEY` isn't set, any API key will do. If a key is set, the client must pass exactly that key — the proxy validates the bearer token before granting access to models, sessions, and completions.

---

## 🔐 Refresh the login

```bash
npm run auth
npm start
```

If DeepSeek starts returning `401`, `403`, or asks for a new PoW/session — run `npm run auth` again and refresh the saved browser session.

Local authorization files must not end up on GitHub:

- `deepseek-auth.json`
- `.chrome-for-testing-profile-deepseek/`
- `.env`

They're already listed in `.gitignore`.

---

## 🧪 Tests

Syntax check, unit tests, and end-to-end tests against a local
DeepSeek mock server (no network or account needed; `node --test` requires
Node.js 18.13+):

```bash
npm test
```

Live smoke tests against a running local proxy (if `PROXY_API_KEY` is set, it's
taken from the environment):

```bash
BASE_URL=http://127.0.0.1:9655 MODEL=deepseek-chat npm run test:live
```

---

## 📌 Project status

FreeDeepseekAPI is an experimental web-chat proxy for local use and integrations. It depends on the current DeepSeek Web Chat contract, so changes on DeepSeek's side may require updates to the auth/session logic or the model mapping.

If something stops working:

1. refresh the login with `npm run auth`;
2. check `/v1/model-capabilities`;
3. retry the request on a fresh session;
4. if the problem persists, DeepSeek has probably changed its internal Web API.

---

<p align="center">
  <strong>ForgetMeAI</strong> · <a href="https://t.me/forgetmeai">Telegram</a>
</p>
