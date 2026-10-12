---
name: cloud-session-read
description: Find and read Devin Cloud session history — resolve session IDs/URLs/titles, pull the full conversation, PRs, status, and artifacts via the devin-cloud MCP server.
---

# Cloud Session Read

Use when the user references a cloud Devin session — `@conversation:` links, `app.devin.ai/sessions/<id>` URLs, session titles, "那个云端会话", "读一下云上那个会话" — and wants its history, status, or deliverables.

## Resolve the session

1. **URL** → the 32-hex id after `/sessions/`.
2. **Title/description** → `devin-cloud` tool `devin_session_search` (filter by title text, status, date).
3. If multiple orgs/accounts exist, a 404 means wrong org — ask the user which account, then they set `DEVIN_ORG_ID` accordingly.

## Pull the record

- `devin_session_events` — full event/message stream (user prompts, devin replies, attachments, tool activity).
- `devin_session_interact` (status) — current state, pull requests, structured output, tags.

## Report faithfully

Preserve: repo + branch names, PR URLs and merge state, verification claims (test counts, Lighthouse, prod checks), explicit pending asks ("要不要放量/收工"), and the last unfinished step. Quote key decisions verbatim rather than paraphrasing.

## Auth check

If calls fail with 401/403: the `DEVIN_API_KEY` / `DEVIN_ORG_ID` env vars aren't set (or `${env:}` isn't expanding on this CLI version). Fix: set the env vars, or add a literal-header `devin-cloud` entry to the user's `mcp_config.json`. Never hardcode the key into any committed file; never echo it.

## Safety

Reading is free. Sending messages, creating, sleeping, or terminating sessions costs ACUs and mutates real state — confirm before calling `devin_session_interact` (send/sleep/terminate) or `devin_session_create`.
