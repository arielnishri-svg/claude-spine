---
name: wrapup
description: Closes a Cowork session by updating all three Drive spine files (NOW, DECISIONS, OPEN) and appending a 3-line summary to the local daily-log.md. Keeps context current so Claude Chat and future Cowork sessions start with accurate state. Triggered when user says "wrap up session", "wrap up", "end session", "close session", or "update the spine".
---

# Wrap Up Session

Update all persistent context files before the user closes Cowork. Do every step in order.

## Step 1 — Summarise this session

Review the current conversation. Extract:
- What was built, decided, or resolved (2–4 bullet points)
- Any locked decisions with their reasoning
- What remains open or unresolved

If the conversation has no meaningful content (e.g. user just tested something), write a minimal summary and note "No major decisions."

## Step 2 — Update NOW.md in Google Drive

Read `memory/global.md` to find the Drive file ID for NOW (stored under `## Drive spine`).

If the ID is found, overwrite the NOW Google Doc with:

```
## NOW — [User name]
Last updated: [today's date]

### What's active today
[One short paragraph — current focus, what's in motion]

### Current projects
[Active projects and their state — one line each]

### System status
[Key connector and skill states — any changes from default]

### Key decisions made recently
[Last 3–5 decisions, most recent first, one line each]

### What's next
[Top 3–5 open items, in priority order]
```

Keep NOW.md to one screen. Cut anything stale.

If no Drive file ID is found in global.md, tell the user: "Drive spine not configured. Say 'set up mission control' to complete setup, or add your Drive file IDs to memory/global.md manually."

## Step 3 — Append to DECISIONS.md in Google Drive

Read `memory/global.md` to find the Drive file ID for DECISIONS.

For each locked decision made this session, append one line:
`[YYYY-MM-DD] — [Decision] — [Reason]`

If no new decisions were made this session, skip this step silently.

## Step 4 — Update OPEN.md in Google Drive

Read `memory/global.md` to find the Drive file ID for OPEN.

- Remove any items that were resolved this session (mark as done or delete)
- Add any new open items surfaced during this session
- Preserve all existing unresolved items

## Step 5 — Write local spine files

Write the same content to local files so the private GitHub repo stays current for Claude Chat:

**File:** `{BASE_PATH}/spine/NOW.md`
Overwrite with the same content written to Drive in Step 2.

**File:** `{BASE_PATH}/spine/DECISIONS.md`
Append the same new decision lines added in Step 3.
If the file does not exist yet, create it and write all decisions.

**File:** `{BASE_PATH}/spine/OPEN.md`
Overwrite with the same content written to Drive in Step 4.

If the `spine/` folder does not exist, create it before writing.

## Step 6 — Append to local daily-log.md

Read `{BASE_PATH}/memory/daily-log.md`.

Append:

```markdown
## [YYYY-MM-DD]
Built/decided: [one line summary]
Key decisions: [one line — most important decision, or "None"]
Next: [one line — highest priority next action]
```

## Step 7 — Confirm and prompt git push

Reply with exactly:

"Session wrapped. Drive spine and local files updated. Run this to sync Claude Chat:
```
cd {BASE_PATH} && git add spine/ memory/daily-log.md && git commit -m 'Wrapup [YYYY-MM-DD]' && git push
```"

---

## Notes

- NOW.md is overwritten every session — it is a pointer, not a record. Old content is replaced.
- DECISIONS.md is append-only — never edit or delete past entries.
- OPEN.md is maintained — completed items are removed, new ones added.
- daily-log.md is append-only — always add at the bottom.
- Local spine files in `spine/` mirror Drive — both updated every session.
- `git push` in the BASE_PATH repo is the step that keeps Claude Chat current.
- If Google Drive is not connected, complete Steps 5, 6, and 7 only, and tell the user which Drive steps were skipped.
