# claude-spine

A context-persistence pattern for Claude. Keeps your state alive when you switch between Claude Cowork, Claude Chat, and Claude Code — without re-explaining anything.

---

## The problem

Every Claude session starts cold. Switch from Cowork to Chat, or open Claude Code in a new terminal, and you lose everything — what you were working on, what you decided, what's still open. You end up spending the first few minutes of every session re-establishing context that should already be there.

---

## The pattern

Three small documents act as a shared memory layer across all Claude tools:

| Doc | What it holds | How it's updated |
|-----|--------------|-----------------|
| **NOW** | Current state — active projects, focus, system status | Overwritten each session |
| **DECISIONS** | Append-only log of locked decisions and their reasoning | Never edited, only appended |
| **OPEN** | Live list of unresolved questions and next steps | Items added and removed as they resolve |

**Cowork writes. Chat reads.**

At the end of every Cowork session, the `wrap up session` skill updates all three docs — in Google Drive and in a local `spine/` folder. A private GitHub repo or a Claude Chat Project linked to these files gives Claude full context before you type a single word.

---

## How to use it

### 1. Install the plugin in Cowork

Download `claude-spine.plugin` from the [latest release](https://github.com/arielnishri-svg/claude-spine/releases/latest) and install it in Cowork.

### 2. Connect your Google services

In Cowork Settings → Connectors:

- **Google Drive** — required for the spine (NOW / DECISIONS / OPEN)
- **Gmail + Google Calendar** — required only if you want the morning brief skill

The spine itself only needs Drive. Connect the others when you're ready.

### 3. Run setup

Say `set up claude-spine` in a new Cowork session. The setup skill:
- Creates your local folder structure and memory files
- Creates the three spine docs (`NOW`, `DECISIONS`, `OPEN`) directly in your Google Drive
- Saves the Drive file IDs to your memory so the wrapup skill can write to them automatically

No manual doc creation needed.

### 4. Connect the spine to Claude Chat

Two options — pick one:

**Option A — GitHub (recommended):**
1. Create a private repo (e.g. `my-spine`)
2. Push your `cowork/` folder to it
3. In Claude Chat → Projects → New Project → Add content → GitHub → connect the repo
4. Claude Chat reads the spine files from GitHub every session

**Option B — Google Drive:**
1. In [claude.ai](https://claude.ai) → Projects → New Project
2. Add content → Google Drive → search for `NOW`, `DECISIONS`, `OPEN`
3. Note: Drive search may not surface newly created files — open each doc in your browser first

### 5. Wrap up every session

Say `wrap up session` before closing Cowork. This updates Drive, writes the local `spine/` files, and prints the git push command if you're using GitHub. This is the habit the whole system depends on. A stale `NOW` means Chat reasons from wrong premises.

---

## What ships with this plugin

The plugin includes five skills as a working reference implementation:

| Say this | What it does | Connectors needed |
|----------|-------------|------------------|
| `set up claude-spine` | One-time setup — folder structure, memory files, Drive spine | Drive |
| `wrap up session` | Updates NOW / DECISIONS / OPEN + local spine/ + daily log | Drive |
| `generate my morning brief` | Pulls Google Calendar + Gmail → structured daily brief | Gmail, Calendar |
| `research AAPL` | Web search + analyst consensus + thesis fit → research note | Web search |
| `check the build queue` | Picks up PRD files from a queue folder and builds them | Depends on PRD |

These are examples. The spine pattern works with any skills you write.

---

## Claude Code compatibility

Add a `CLAUDE.md` at the root of any repo:

```
Read ~/Documents/Claude\ Cowork/cowork/memory/global.md for user context and active projects.
Read ~/Documents/Claude\ Cowork/cowork/spine/NOW.md for current state.
```

Local spine files are updated by `wrap up session` and synced via `git push` — they reflect the last completed session.

---

## The spine protocol

If you want to implement the spine pattern without this plugin, the rules are:

- `NOW` is a pointer, not a record. It holds current state only — overwrite it completely each session.
- `DECISIONS` is append-only. Date, decision, reason — one line per entry. Never edit past entries.
- `OPEN` is a live list. Add items when they surface, remove them when they resolve.
- Store the Drive file IDs somewhere your wrapup skill can read them (e.g. `memory/global.md` under `## Drive spine`).
- Keep a local mirror of the spine files and push them to a private GitHub repo — Drive search is unreliable for newly created files.
- The tool that writes the spine (Cowork) and the tool that reads it (Chat) must agree on the format.

---

## License

MIT. Fork it, adapt it, build your own spine.
