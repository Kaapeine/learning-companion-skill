# Learning Companion

<!-- Before publishing: replace REPLACE_ME_GITHUB_USER in learning-companion/SKILL.md and REPLACE_ME_YOUR_NAME in LICENSE. -->

A Claude skill for long-running, multi-session learning — where **you write the textbook**.

You bring a subject (a topic list, a textbook index, or just a name). The AI completes the picture, calibrates to your level, builds a syllabus, and teaches Socratically — sketching each concept's core, then making you do the thinking through predictions, edge cases, and connections. At natural stopping points it proposes a checkpoint: it recaps, **you write the entry in your own words**, it gives honest feedback, you revise, you commit. Your committed text goes into the document verbatim and is never touched by the AI again — not even typos. Over weeks, the document becomes a textbook of the subject written entirely in your own voice, in the order you actually learned it.

## The document

Everything lives in **one self-contained HTML file** — the single source of truth and your save state:

- **Syllabus** — a guide, not a rail; tangents get grafted in where they happened
- **Concept map** — a Mermaid graph of topic names and links, grown as you learn
- **Textbook** — your committed entries, in learning order, with git-style commit lines; optionally illustrated with AI-made figures (inline SVG, Mermaid, or self-contained interactive widgets) in clearly labeled blocks below your text
- **Session log** — append-only, a few lines per session
- **AI state** (collapsed) — a handoff block one AI instance writes for the next: position, open threads, misconceptions to watch, and a learner profile that refines forever and never resets
- **AI instructions** (collapsed) — the full resume protocol, baked into the file

Because the instructions travel inside the file, the document is **self-bootstrapping**: upload it to any Claude conversation and say "resume my learning" — it works even without this skill installed. Share a document with a friend and it carries everything they need.

The file is also **directly editable**: an Edit/Save toolbar lets you edit your own entries in the browser. Save writes the file back in place on desktop Chrome/Edge (File System Access API) and downloads a fresh copy everywhere else. Saving is idempotent — Mermaid sources and widget runtime state are never baked into the file.

## Install

**claude.ai** (Pro plan or higher): Settings → Capabilities → enable *Code execution and file creation*, then Customize → Skills → Upload skill → select the packaged zip (rename `learning-companion.skill` to `.zip` if the picker insists).

**Claude Code**: unzip into `~/.claude/skills/learning-companion/` (personal) or `.claude/skills/learning-companion/` (per project).

## Use

- **Start:** "Start a learning session — here are my topics: …" (naming the skill on the first run guarantees it triggers)
- **During:** tangents are expected; "just tell me" is always honored; checkpoints are pauses, not tests
- **After each checkpoint in claude.ai:** download the updated document — the file is your save state, the chat is disposable. (In Claude Code the file is edited in place.)
- **Resume:** new chat, attach the document, "resume my learning". Attach the companion `<subject>-transcript.md` too if you want one continuous conversation log.

## Design principles

1. **The learner writes everything that gets committed.** Rewriting in your own words is where the learning happens; the AI's recap is a refresher, never a model answer to transcribe.
2. **Committed text is immutable to the AI.** Errors are corrected by *new* entries; the record of your understanding at each point in time is preserved. (You can edit your own entries freely — the rule binds the AI, not you.)
3. **One file, no sync.** State, content, instructions, and editor all travel together; document and protocol can never drift apart.
4. **Tangents are features.** They're tracked, checkpointed, and grafted into the syllabus and map rather than evaporating.

## Files

- `SKILL.md` — the skill (setup protocol, teaching, checkpoints, persistence)
- `assets/learning-doc-template.html` — the document template (format v1.0)
