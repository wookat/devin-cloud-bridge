# devin-cloud-bridge

A [Devin CLI](https://devin.ai) plugin that bridges local sessions with **Devin Cloud**: read cloud session history, pull a cloud session's plan down and continue it locally, and message/spawn cloud sessions — all via the official Devin MCP endpoint (`mcp.devin.ai`).

## Why

Cloud Devin sessions hold their plans, decisions, and progress inside `app.devin.ai`. The local desktop/CLI agent can't see them by default. This plugin makes the local agent cloud-aware so you can:

- **Read** any cloud session's full conversation ("那个搭建 SEO 页面体系的会话干了什么？")
- **Handoff** — move a cloud session's plan to local execution and keep working it locally
- **Message/spawn** cloud sessions without opening the web app

## What's inside

```
.devin-plugin/plugin.json   # manifest + MCP server declaration
.mcp.json                   # (conventional root MCP config — same declaration)
skills/
  cloud-session-read/       # resolve + read cloud sessions, report faithfully
  cloud-handoff/            # cloud→local handoff workflow: brief → clone → resume
```

The plugin wires the **official Devin MCP server** (`https://mcp.devin.ai/mcp`) into your session with your API key, unlocking the private tool set:

`devin_session_search` · `devin_session_events` · `devin_session_interact` (send/sleep/terminate) · `devin_session_create` · `devin_session_gather` · `devin_session_insights` · plus knowledge / playbook / schedule / automation / review / code-scan management.

## Setup

### 1. Get credentials

- **API key** — <https://app.devin.ai> → Settings → Devin API → create a personal access token (`cog_…`), or an org-scoped service-user key.
- **Org ID** — same page, format `org-<32hex>`.

### 2. Provide credentials

**Option A — env vars (recommended):**

```bash
export DEVIN_API_KEY='cog_...'
export DEVIN_ORG_ID='org-...'
```

The bundled MCP config expands `${env:DEVIN_API_KEY}` / `${env:DEVIN_ORG_ID}` in request headers.

**Option B — literal headers in your user config** (works on every CLI version, including ones without header env-expansion). Add to `~/.config/devin/mcp_config.json` (`%APPDATA%\devin\mcp_config.json` on Windows):

```json
"devin-cloud": {
  "url": "https://mcp.devin.ai/mcp",
  "transport": "http",
  "headers": {
    "Authorization": "Bearer cog_...",
    "X-Org-Id": "org-..."
  }
}
```

> Never commit your key to a repo. Option B keeps it in your machine-local config only.

### 3. Install the plugin

```bash
devin plugins install --local /path/to/devin-cloud-bridge   # local link, this machine
# or from GitHub once published:
devin plugins install wookat/devin-cloud-bridge
```

New sessions then expose `mcp__devin-cloud__*` tools and the skills `/devin-cloud-bridge:cloud-session-read` and `/devin-cloud-bridge:cloud-handoff` (also auto-triggered when you paste a session URL or ask about a cloud conversation).

## Use it

- "读一下这个云端会话 https://app.devin.ai/sessions/<id>" → agent pulls the full record.
- "云端那个 SEO 会话搬到本地继续" → handoff flow: brief → clone/checkout → resume with your confirmation.
- "给 528bbbd2 会话发条消息让它收工" → `devin_session_interact` (asks before mutating).

## Notes

- Reading is free; **sending/creating/terminating sessions costs ACU** — the skills always confirm first.
- Sessions live in a specific org; a 404 usually means the wrong `DEVIN_ORG_ID`.
- No affiliation with Cognition AI; uses only the [documented public API](https://docs.devin.ai/api-reference) and the official MCP endpoint.

## License

MIT
