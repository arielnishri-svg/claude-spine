# claude-spine

A context-persistence pattern for Claude. Keeps your state alive when you switch between Claude Cowork, Claude Chat and Claude Code, without re-explaining anything.

---

## The problem

Every Claude session starts cold. Switch from Cowork to Chat, or open Claude Code in a new terminal, and you lose everything: what you were working on, what you decided, what's still open. You spend the first few minutes of every session re-establishing context that should already be there.

---

## The pattern

Three small documents act as a shared memory layer across all Claude tools:

| Doc | What it holds | How it's updated |
|-----|--------------|-----------------|
| **NOW** | Current state: active projects, focus, system status | Overwritten each session |
| **DECISIONS** | Append-only log of locked decisions and their reasoning | Never edited, only appended |
| **OPEN** | Live list of unresolved questions and next steps | Items added and removed as they resolve |

**Cowork writes. Chat reads.**

At the end of every Cowork session, `wrap up session` updates all three docs in Google Drive and in a local `spine/` folder. A private GitHub repo or a Claude Chat Project linked to these files gives Claude full context before you type a single word.

---

## How to use it

### 1. Install the plugin in Cowork

Download `claude-spine.plugin` from the [latest release](https://github.com/arielnishri-svg/claude-spine/releases/latest) and install it in Cowork.

### 2. Connect Google Drive

In Cowork Settings, Connectors, connect **Google Drive**. This is the only connector the spine itself requires.

### 3. Run setup

Say `set up claude-spine` in a new Cowork session. The setup skill:
- Creates your local folder structure and memory files
- Creates the three spine docs (`NOW`, `DECISIONS`, `OPEN`) in your Google Drive
- Saves the Drive file IDs to memory so the wrapup skill can write to them automatically

No manual doc creation needed.

### 4. Connect the spine to Claude Chat

Two options. Pick one:

**Option A: GitHub (recommended)**
1. Create a private repo (e.g. `my-spine`)
2. Push your local spine folder to it
3. In Claude Chat, go to Projects, New Project, Add content, GitHub and connect the repo
4. Claude Chat reads your spine files every session

**Option B: Google Drive**
1. Go to [claude.ai](https://claude.ai), Projects, New Project
2. Add content, Google Drive, search for `NOW`, `DECISIONS`, `OPEN`
3. Note: Drive search may not surface newly created files. Open each doc in your browser first.

### 5. Wrap up every session

Say `wrap up session` before closing Cowork. This updates Drive, writes the local `spine/` files and prints the git push command if you're using GitHub.

This is the habit the whole system depends on. A stale `NOW` means Chat reasons from wrong premises.

---

## What ships with this plugin

Two skills, the minimum required to run the spine:

| Say this | What it does | Connectors needed |
|----------|-------------|------------------|
| `set up claude-spine` | One-time setup: folder structure, memory files, Drive spine | Drive |
| `wrap up session` | Updates NOW / DECISIONS / OPEN + local spine/ + daily log | Drive |

Everything else (morning briefs, investment research, task automation) is your layer to build on top.

---

## Claude Code compatibility

Add a `CLAUDE.md` at the root of any repo:

```
Read ~/Documents/Claude\ Cowork/cowork/memory/global.md for user context and active projects.
Read ~/Documents/Claude\ Cowork/cowork/spine/NOW.md for current state.
```

Local spine files are updated by `wrap up session` and synced via `git push`. They reflect the last completed session.

---

## The spine protocol

If you want to implement this pattern without the plugin:

- `NOW` is a pointer, not a record. It holds current state only. Overwrite it completely each session.
- `DECISIONS` is append-only. Date, decision, reason. Never edit past entries.
- `OPEN` is a live list. Add items when they surface, remove them when they resolve.
- Store the Drive file IDs somewhere your wrapup skill can read them (e.g. `memory/global.md` under `## Drive spine`).
- Keep a local mirror and push to a private GitHub repo. Drive search is unreliable for newly created files.
- The tool that writes the spine (Cowork) and the tool that reads it (Chat) must agree on the format.

---

## License

MIT. Fork it, adapt it, build your own spine.
