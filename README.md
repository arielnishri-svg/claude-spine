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

At the end of every Cowork session, the `wrap up session` skill updates all three docs. In Claude Chat, a Project linked to these Drive files gives Opus full context before you type a single word. Claude Code can read the local copies directly.

---

## How to use it

### 1. Create the three docs in Google Drive

Create three Google Docs named `NOW`, `DECISIONS`, and `OPEN`. Keep them in a folder called `_spine`.

### 2. Install this plugin in Cowork

Install the `.plugin` file in Cowork (Settings → Plugins → Install from file). Connect Gmail, Google Calendar, and Google Drive in Settings → Connectors.

### 3. Run setup

Say `set up mission control` in a new Cowork session. The setup skill creates your folder structure, seeds your memory files, and links the Drive spine docs.

### 4. Connect Drive to Claude Chat

In [claude.ai](https://claude.ai) → Projects → New Project:
1. Add content → Google Drive
2. Add the `NOW`, `DECISIONS`, and `OPEN` docs
3. Use this project for any Chat session where you want context continuity

### 5. Wrap up every session

Say `wrap up session` before closing Cowork. This is the habit the whole system depends on. A stale `NOW` means Chat reasons from wrong premises.

---

## What ships with this plugin

The plugin includes five skills as a working reference implementation:

| Say this | What it does |
|----------|-------------|
| `set up mission control` | One-time setup — folder structure, memory files, Drive spine |
| `generate my morning brief` | Pulls Google Calendar + Gmail → structured daily brief |
| `research AAPL` | Web search + analyst consensus + thesis fit → research note |
| `wrap up session` | Updates NOW / DECISIONS / OPEN + appends to daily log |
| `check the build queue` | Picks up PRD files from a queue folder and builds them |

These are examples. The spine pattern works with any skills you write.

---

## Claude Code compatibility

The spine docs are Google Docs, but Cowork also maintains local markdown copies in your workspace. Claude Code can read those directly. Add a `CLAUDE.md` at the root of any repo pointing at `memory/global.md` and your daily brief — Claude Code picks it up automatically at session start.

---

## The spine protocol

If you want to implement the spine pattern without this plugin, the rules are:

- `NOW` is a pointer, not a record. It holds current state only — overwrite it completely each session.
- `DECISIONS` is append-only. Date, decision, reason — one line per entry. Never edit past entries.
- `OPEN` is a live list. Add items when they surface, remove them when they resolve.
- The tool that writes the spine (Cowork) and the tool that reads it (Chat) must agree on the format.

---

## License

MIT. Fork it, adapt it, build your own spine.
