# Claude Spine

A context-persistence pattern for Claude. Keeps your state alive when you switch between Claude Cowork, Claude Chat, and Claude Code — without re-explaining anything.

---

## The problem

Every Claude session starts cold. Switch from Cowork to Chat, or open Claude Code in a new terminal, and you lose everything — what you were working on, what you decided, what's still open. You end up spending the first few minutes of every session re-establishing context that should already be there.

---

## The pattern

Three small Google Drive documents act as a shared memory layer across all Claude tools:

| Doc | What it holds | How it's updated |
|-----|--------------|-----------------|
| **NOW** | Current state — active projects, focus, system status | Overwritten each session |
| **DECISIONS** | Append-only log of locked decisions and their reasoning | Never edited, only appended |
| **OPEN** | Live list of unresolved questions and next steps | Items added and removed as they resolve |

**Cowork writes. Chat reads.**

At the end of every Cowork session, the `wrap up session` skill updates all three docs in Google Drive. In Claude Chat, a Project linked to these Drive files gives Opus full context before you type a single word.

---

## How to use it

### 1. Install the plugin in Cowork

Download `claude-spine.plugin` from the [latest release](https://github.com/arielnishri-svg/claude-spine/releases/latest) and install it in Cowork (Settings → Plugins → Install from file).

### 2. Connect your Google services

In Cowork Settings → Connectors:

- **Google Drive** — required for the spine (NOW / DECISIONS / OPEN)
- **Gmail + Google Calendar** — required only if you want the morning brief skill

The spine itself only needs Drive. Connect the others when you're ready.

### 3. Run setup

Say `set up mission control` in a new Cowork session. The setup skill:
- Creates your local folder structure and memory files
- Creates the three spine docs (`NOW`, `DECISIONS`, `OPEN`) directly in your Google Drive
- Saves the Drive file IDs to your memory so the wrapup skill can write to them automatically

No manual doc creation needed.

### 4. Connect Drive to Claude Chat

In [claude.ai](https://claude.ai) → Projects → New Project:
1. Open each spine doc in your browser first (Drive search in Claude Chat may not surface newly created files until they've been viewed)
2. Add content → Google Drive → search for `NOW`, `DECISIONS`, `OPEN`
3. Use this project for any Chat session where you want context continuity

### 5. Wrap up every session

Say `wrap up session` before closing Cowork. This is the habit the whole system depends on. A stale `NOW` means Chat reasons from wrong premises.

---

## What ships with this plugin

The plugin includes five skills as a working reference implementation:

| Say this | What it does | Connectors needed |
|----------|-------------|------------------|
| `set up mission control` | One-time setup — folder structure, memory files, Drive spine | Drive |
| `wrap up session` | Updates NOW / DECISIONS / OPEN + appends to daily log | Drive |
| `generate my morning brief` | Pulls Google Calendar + Gmail → structured daily brief | Gmail, Calendar |
| `research AAPL` | Web search + analyst consensus + thesis fit → research note | Web search |
| `check the build queue` | Picks up PRD files from a queue folder and builds them | Depends on PRD |

These are examples. The spine pattern works with any skills you write.

---

## Claude Code compatibility

The spine docs live in Google Drive and are updated by Cowork's wrapup skill. Claude Code can't read Drive directly, but you can point it at the local memory files in your workspace. Add a `CLAUDE.md` at the root of any repo with a line like:

```
Read ~/Documents/Claude\ Cowork/cowork/memory/global.md for user context and active projects.
```

Note: local memory files only reflect the last time you ran `wrap up session` in Cowork — they are not synced automatically.

---

## The spine protocol

If you want to implement the spine pattern without this plugin, the rules are:

- `NOW` is a pointer, not a record. It holds current state only — overwrite it completely each session.
- `DECISIONS` is append-only. Date, decision, reason — one line per entry. Never edit past entries.
- `OPEN` is a live list. Add items when they surface, remove them when they resolve.
- Store the Drive file IDs somewhere your wrapup skill can read them (e.g. a local memory file). The skill needs the IDs to write — not just the doc names.
- The tool that writes the spine (Cowork) and the tool that reads it (Chat) must agree on the format.

---

## License

MIT. Fork it, adapt it, build your own spine.
