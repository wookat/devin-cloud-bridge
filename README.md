# devin-cloud-bridge

**English** · [简体中文](./README.zh-CN.md)

<p>
  <img alt="License: MIT" src="https://img.shields.io/badge/license-MIT-blue.svg">
  <img alt="MCP" src="https://img.shields.io/badge/MCP-compatible-green">
  <img alt="Devin API v3" src="https://img.shields.io/badge/Devin_API-v3-7c3aed">
  <img alt="PRs welcome" src="https://img.shields.io/badge/PRs-welcome-brightgreen.svg">
</p>

A [Devin CLI](https://devin.ai) plugin that bridges **local sessions ↔ Devin Cloud**: read cloud session history, pull a cloud session's plan down and **continue it locally**, and message/spawn cloud sessions — powered by the official Devin MCP endpoint (`mcp.devin.ai`).

## 🌉 Why

Cloud Devin sessions keep their plans, decisions, and progress inside `app.devin.ai`. A local desktop/CLI agent can't see them by default. This plugin makes your local agent cloud-aware so you can:

- 🔍 **Read** any cloud session's full conversation ("what did that SEO-pages session actually do?")
- 📦 **Handoff** — move a cloud session's plan to local execution and keep working it locally
- 💬 **Message / spawn** cloud sessions without opening the web app

## 📁 What's inside

```
.devin-plugin/plugin.json   # manifest + MCP server declaration
.mcp.json                   # conventional root MCP config (works with other MCP clients too)
skills/
  cloud-session-read/       # resolve + read cloud sessions, report faithfully
  cloud-handoff/            # cloud→local handoff: brief → clone → resume
```

The plugin wires the **official Devin MCP server** into your session with your API key, unlocking the private tool set:

| Area | Tools |
|---|---|
| 🗂️ Sessions | `devin_session_search` · `devin_session_events` · `devin_session_interact` (send/sleep/terminate) · `devin_session_create` · `devin_session_gather` · `devin_session_insights` |
| 🧠 Platform | `devin_knowledge_manage` · `devin_playbook_manage` · `devin_schedule_manage` · `devin_automation_manage` |
| 🛡️ Ops | `devin_review_manage` · `devin_code_scan_manage` · `devin_oncall_manage` · `devin_billing_tag_manage` |

## 🚀 Setup

### 1. Get credentials

- **API key** — <https://app.devin.ai> → **Settings → Devin API** → create a personal access token (`cog_…`), or an org-scoped service-user key.
- **Org ID** — same page; format `org-<32hex>`.

### 2. Provide credentials

**Option A — credential placeholders (recommended):** the manifest declares `Authorization: Bearer ${DEVIN_API_KEY}` and `X-Org-Id: ${DEVIN_ORG_ID}`. In Devin Cloud, open the plugin's **Connect** flow and supply both values; locally, export them as env vars:

```bash
export DEVIN_API_KEY='cog_...'
export DEVIN_ORG_ID='org-...'
```

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

> ⚠️ Never commit your key to a repo. Both options keep it in your local environment only.

### 3. Install the plugin

```bash
devin plugins install wookat/devin-cloud-bridge
# or, for development:
devin plugins install --local /path/to/devin-cloud-bridge
```

New sessions then expose `mcp__devin-cloud__*` tools plus the skills `/devin-cloud-bridge:cloud-session-read` and `/devin-cloud-bridge:cloud-handoff` (they also auto-trigger when you paste a session URL or ask about a cloud conversation).

### 4. One-prompt install (send to any Devin session)

Instead of installing by hand, paste this prompt into any local or cloud Devin session — the agent installs and configures it for you:

```text
Install the plugin https://github.com/wookat/devin-cloud-bridge — run
`devin plugins install wookat/devin-cloud-bridge`, then ask me for my
DEVIN_API_KEY (a `cog_` token) and DEVIN_ORG_ID (`org-…`), write them as
literal headers into a `devin-cloud` entry in my user mcp_config.json
(never commit them), and verify by calling tools/list on the new server.
```

The agent may still need one `devin auth login` (browser) for the `plugins install` command — that's a one-time CLI login, unrelated to the API key.

## 👥 Multiple accounts / orgs

One `devin-cloud` entry covers one org. For multiple Devin accounts, add one entry per account — same URL, different credentials, named `devin-cloud-<label>`:

```json
"devin-cloud-acc1": { "url": "https://mcp.devin.ai/mcp", "transport": "http",
  "headers": { "Authorization": "Bearer cog_acc1...", "X-Org-Id": "org-acc1..." } },
"devin-cloud-acc2": { "url": "https://mcp.devin.ai/mcp", "transport": "http",
  "headers": { "Authorization": "Bearer cog_acc2...", "X-Org-Id": "org-acc2..." } }
```

Tools are namespaced per server (`mcp__devin-cloud-acc1__*` vs `mcp__devin-cloud-acc2__*`), so one local session can read — and act on — every account's cloud sessions. Sessions can't move between orgs, but the plan can: read events on account A, then `devin_session_create` on account B with the handoff brief as the prompt (the `cloud-handoff` skill covers this flow).

## 🧭 Usage

- **"Read this cloud session: https://app.devin.ai/sessions/\<id\>"** → the agent pulls the full record.
- **"Move that cloud session's plan to local and keep going"** → handoff flow: brief → clone/checkout → resume, with your confirmation at each step.
- **"Resume cloud session X on my other account"** → read on A → create on B.
- **"Tell session `528bbbd2…` to wrap up"** → `devin_session_interact` (the agent confirms before mutating).

## ⚠️ Notes

- Reading is free; **sending/creating/terminating sessions consumes ACU** — the skills always confirm first.
- Sessions live in a specific org; a 404 usually means the wrong `DEVIN_ORG_ID`, not a missing session.
- Cloud-VM-only changes (uncommitted, unpushed) can't be recovered locally — the handoff skill calls this out explicitly when it matters.
- Unofficial community plugin; built on the [documented public API](https://docs.devin.ai/api-reference) and the official MCP endpoint. Not affiliated with Cognition AI.

## 📄 License

[MIT](./LICENSE)
