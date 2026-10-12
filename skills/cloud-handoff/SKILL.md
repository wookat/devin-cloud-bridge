---
name: cloud-handoff
description: Pull a Devin Cloud session's plan and progress down to this machine and continue the work locally — extract context, clone the repo, reconstruct state, resume execution.
---

# Cloud → Local Handoff

Use when the user wants to stop working in a cloud Devin session and continue its plan locally ("把云端会话搬到本地", "云端那个规划本地接着做", "handoff to local").

## Workflow

### 1. Read the cloud session

Follow the `cloud-session-read` flow: resolve the session, pull `devin_session_events` (full history) and session status (PRs, structured output).

### 2. Build the handoff brief

From the messages extract, in order:

- **Original task** — the first user prompt verbatim.
- **Done + verified** — merged PRs, deployed URLs, test/CI results the session reported.
- **Pending plan** — the session's own proposed next steps (sessions usually end with "要不要…/下一步…" proposals); list them as options, don't silently pick.
- **Key paths/decisions** — files, data models, conventions discovered (e.g. "OCCUPATIONS in v2/functions/_seo/data.ts").
- **Open issues** — bugs found, fixes in flight, blockers.
- **Parent/child sessions** — check `parent_session_id` / `child_session_ids`; a crashed parent may hold earlier context worth pulling too.

### 3. Materialize locally

- Identify the repo(s) from PR URLs (`github.com/<owner>/<repo>/pull/…`) and clone into the user's projects dir if absent; otherwise `git fetch` + checkout the state the session left (usually latest `main`, or the PR head if unmerged).
- Unmerged PR branches: `gh pr checkout <n>` or fetch `pull/<n>/head`.
- Cloud-VM-only changes (uncommitted) are unreachable — if events suggest unpushed work, say so honestly.

### 4. Resume

- Present the brief + recommended next action and **confirm with the user before mutating** (the cloud session's proposal isn't automatically approved).
- Consider writing the brief to the repo's handoff doc (`docs/handoff/`, `状态.md`, or `HANDOFF.md`) if the project has that convention — it becomes the next session's context.

### 5. Settle the cloud side (ask first — costs ACU/state)

- Leave suspended (default): free, resumable later.
- `devin_session_interact` → send a "work continues locally, sleep" note if the user wants a record.
- Terminate only on explicit request.

## Pitfalls

- Cloud session events are the source of truth — repo state on GitHub may lag behind session claims; verify `git log` and PR merge state before trusting "已合并/已上线".
- Attachments (screenshots/videos) aren't needed for continuation; mention them but don't try to download.
- Wrong-org 404s: the session lives under a specific `X-Org-Id`; if the user runs multiple Devin accounts, confirm which org the session is in.
