<p align="center">
<svg width="480" height="140" viewBox="0 0 480 140" xmlns="http://www.w3.org/2000/svg">
  <path d="M30 90 Q70 20 110 70 T190 60"
        fill="none"
        stroke="#6366f1"
        stroke-width="3"
        stroke-linecap="round"/>
  <circle cx="85" cy="55" r="3" fill="#6366f1"/>
  <text x="210" y="65"
        font-family="Segoe UI, sans-serif"
        font-size="36"
        font-weight="700"
        fill="currentColor">
        Wyrmling
  </text>
  <text x="210" y="90"
        font-family="Segoe UI, sans-serif"
        font-size="14"
        font-weight="500"
        fill="currentColor">
        Autonomous AI Agent
  </text>
</svg>
</p>

<br/>

# Wyrmling - Autonomous AI Agent

A full-fledged autonomous agent built on Dragon programming language, multi-provider
(Ollama / OpenAI / Anthropic / Groq / DeepSeek / LM Studio / any OpenAI-compatible),
config-driven, with a modern responsive web UI.

---

## File layout

```
wyrm/
├── main.dgn              # Entry point - run this
├── config.dgn            # Loads / saves / queries config.json
├── config.json           # All settings, providers, tool toggles
├── logger.dgn            # ANSI-colored console logging
├── memory.dgn            # Persistent core-memory (wyrm_memory.json)
├── sessions.dgn          # Multi-conversation persistence (wyrm_sessions.json)
├── agent.dgn             # The autonomous loop — calls provider, runs tools
├── server.dgn            # HTTP server + REST routes
├── ui.dgn                # Generates the responsive single-page UI
│
├── providers/
│   ├── router.dgn        # Dispatches to the right provider
│   ├── ollama.dgn        # /api/chat for Ollama
│   ├── openai.dgn        # /v1/chat/completions (OpenAI + compatibles)
│   ├── deepseek.dgn      # for deepseek
│   └── anthropic.dgn     # /v1/messages (Claude)
│
└── tools/
    └── registry.dgn      # All tool schemas + implementations + dispatcher
```

Generated at runtime:
- `wyrm_ui.html` — emitted by `ui.dgn` on first boot
- `wyrm_memory.json` — core memories
- `wyrm_sessions.json` — chat history per session

---

## Run

```
dragon main.dgn
```

Then open `http://localhost:8080` (or whatever port you set in `config.json`).

### Login

The web UI is protected by a login screen. The default credentials are:

```
username: admin
password: admin
```

You can change the password anytime from **Settings → Security**. Change it from
the default before exposing the server on any untrusted network.

---

## What it can do

The agent has 11 tools out of the box — every one of them is enable/disable-able
from the Settings UI:

| Tool | What it does |
|------|---|
| `run_shell`          | Execute any shell/cmd command |
| `run_python`         | Execute Python source code |
| `read_file` / `write_file` / `delete_file` / `list_dir` | Filesystem |
| `web_search`         | DuckDuckGo HTML search |
| `fetch_url`          | Fetch a URL, strip HTML, return text |
| `core_memory_append/remove` | Persistent memory across sessions |
| `get_time`           | Current local time |
| `calculator`         | Safe math expression evaluation |

---

## Providers

Switch providers from the UI's Settings → Provider tab. Defaults:

| Provider | Endpoint | Notes |
|----------|----------|-------|
| Ollama | `http://localhost:11434` | Local, no API key |
| OpenAI | `https://api.openai.com/v1` | Set API key |
| Anthropic | `https://api.anthropic.com/v1` | Set API key |
| Groq | `https://api.groq.com/openai/v1` | OpenAI-compatible |
| DeepSeek | `https://api.deepseek.com/` | Set API Key |
| LM Studio | `http://localhost:1234/v1` | Local, no key |
| Custom | configurable | For OpenRouter, Together, vLLM, etc. |

Adding a brand-new provider is one entry in `config.json`'s `providers` map
(set `kind` to `"openai"`, `"ollama"`, `deepseek` , or `"anthropic"`).

---

## REST API

| Method | Path | Body | Purpose |
|---|---|---|---|
| `GET`    | `/`                    | — | Serves the chat UI |
| `POST`   | `/api/chat`            | `{session_id, message}` | Run agent for one turn |
| `GET`    | `/api/sessions`        | — | List all sessions |
| `POST`   | `/api/sessions`        | — | Create session |
| `GET`    | `/api/sessions/:id`    | — | Get session messages |
| `DELETE` | `/api/sessions/:id`    | — | Delete a session |
| `GET`    | `/api/settings`        | — | Get full config |
| `POST`   | `/api/settings`        | `{config}` | Save config |
| `POST`   | `/api/login`           | `{username, password}` | Authenticate, returns token |
| `POST`   | `/api/change-password` | `{current_password, new_password}` | Change the login password |
| `GET`    | `/api/memory`          | — | List core memories |
| `POST`   | `/api/memory/clear`    | — | Wipe all memories |
| `GET`    | `/api/health`          | — | Health check |

---

## UI features

- **Sidebar** with all chats, click to switch, hover to delete
- **Markdown rendering** of agent replies (code fences, bold, italic, lists, links)
- **Tool-call cards** that collapse/expand to show args + result
- **Settings modal** with 5 tabs:
  - *Provider* - pick provider, set base URL/model/API key
  - *Behavior* - temperature, max tokens, max steps, context window, extra system prompt
  - *Tools* - enable/disable individual tools
  - *Appearance* - light / dark theme
  - *Security* - change the login password
- **Auto-resizing textarea** with Enter-to-send / Shift+Enter for newline
- **Responsive** - full sidebar on desktop, slide-over on mobile
- **Live status pill** showing the active provider + model
