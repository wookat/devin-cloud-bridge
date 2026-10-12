---
name: cloud-session-read
description: Find and read Devin Cloud session history — resolve session IDs/URLs/titles, pull the full conversation, PRs, status, and artifacts via the devin-cloud MCP server.
---

# Cloud Session Read

Use when the user references a cloud Devin session — `@conversation:` links, `app.devin.ai/sessions/<id>` URLs, session titles, "那个云端会话", "读一下云上那个会话" — and wants its history, status, or deliverables.

## Resolve the session

1. **URL** → the 32-hex id after `/sessions/`.
2. **Title/description** → `devin-cloud` tool `devin_session_search` (filter by title text, status, date).
3. A 404 means wrong org — the session lives under another `X-Org-Id`.

## Multiple accounts / orgs

Each Devin account (org) needs its own PAT, and each gets its own MCP server entry named `devin-cloud-<label>` (e.g. `devin-cloud-acc1`, `devin-cloud-acc2`) in `mcp_config.json`. The plugin installs one entry; extra accounts are extra entries with the same `url` and different `Authorization`/`X-Org-Id` headers.

- Tools appear namespaced per server: `mcp__devin-cloud-acc1__devin_session_search`, etc.
- If the user doesn't say which account a session is in, search the default `devin-cloud` first, then each additional account until found.
- Sessions belong to orgs, not browser profiles — one PAT sees every session in its org.

## Pull the record

- `devin_session_events` — full event/message stream (user prompts, devin replies, attachments, tool activity).
- `devin_session_interact` (status) — current state, pull requests, structured output, tags.

## Report faithfully

Preserve: repo + branch names, PR URLs and merge state, verification claims (test counts, Lighthouse, prod checks), explicit pending asks ("要不要放量/收工"), and the last unfinished step. Quote key decisions verbatim rather than paraphrasing.

## Auth check

If calls fail with 401/403: the `DEVIN_API_KEY` / `DEVIN_ORG_ID` env vars aren't set (or `${env:}` isn't expanding on this CLI version). Fix: set the env vars, or add a literal-header `devin-cloud` entry to the user's `mcp_config.json`. Never hardcode the key into any committed file; never echo it.

## Safety

Reading is free. Sending messages, creating, sleeping, or terminating sessions costs ACUs and mutates real state — confirm before calling `devin_session_interact` (send/sleep/terminate) or `devin_session_create`.
