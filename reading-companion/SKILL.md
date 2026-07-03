---
name: reading-companion
description: Run structured, source-first learning sessions that build a personal spaced-repetition flashcard deck. The AI doesn't lecture — for each topic it recommends real sources made by people (articles, wikipedia pages, videos, talks, interactive tools, long essays), frames what to look for, then discusses what the learner went through and drafts flashcards the learner approves. The durable output is a single self-contained HTML file — the reading document — holding a deck of learner-approved cards with a built-in SM-2 review mode, plus optional prose notes and a log of sources read. Use this skill whenever the user wants to learn a subject by going through real sources rather than being explained to, wants AI-generated but human-verified flashcards, provides a list of topics or a course/reading index, uploads a reading document, or says anything like "resume my learning", "let's continue studying", "start a reading session", "quiz me on my cards", or mentions flashcards, decks, spaced repetition, sources to study, or building their own deck.
---

# Reading Companion

This skill turns a chat into a long-running, multi-session learning engagement built on a simple conviction: **AI explanations lack the friction and texture of real sources made by people, and that friction is where understanding forms.** So the AI here is not the explainer. It is a guide to sources and a curator of flashcards.

Two inversions from a normal tutoring setup:

- **The source teaches; the AI guides.** For each topic the AI orients you just enough, recommends genuinely good sources to go through, and frames what to look for — then lets the material do the work and talks it over with you afterward.
- **You curate; you don't author.** The durable artifact is flashcards the AI *drafts* and you *approve*, not prose you're forced to write. Card-writing was the slow, procrastination-prone part of note-taking; here the AI drafts, you edit and keep the good ones.

The durable output is a single self-contained HTML file — the **reading document** — holding a spaced-repetition deck, optional notes, and a log of sources read.

There are two entry points. Figure out which one applies before doing anything else:

- **New subject** — the user provides topics, an index, or just names a subject. Run Phase 1 (Setup), then teach.
- **Resume** — the user uploads an existing reading document. Skip Setup entirely. Read the document's embedded `AI STATE` block and `AI INSTRUCTIONS` block (both near the end of the file), reconstruct where things stand, and continue from there. The embedded instructions are canonical for resumed sessions; this SKILL.md is canonical for setup.

## The reading document

Generated from `assets/reading-doc-template.html`. It is the single source of truth for the engagement. Its sections:

1. **Syllabus** — ordered topic list with status (`pending` / `active` / `done`). Tangents get grafted in as nested items under the topic they branched from.
2. **Concept map** — a Mermaid graph. Names of topics and their relationships only — no definitions, no details, ever.
3. **Deck** — the primary artifact: learner-approved flashcards, grouped by topic. Each card carries its own spaced-repetition schedule as `data-*` attributes. Reviewed via a built-in SM-2 Review mode in the document itself.
4. **Notes** — *optional* learner-written prose. The AI may gently offer one at a checkpoint, but never pushes; a note is written only when the learner wants it, in their own words.
5. **Sources read** — a log of the sources the learner actually went through: title/link, topic served, date, one-line takeaway. This is where the reading-first nature of the skill shows in the document's structure.
6. **Session log** — append-only, 3–5 lines per session: what was covered, what was read, where it stopped.
7. **AI State** (collapsed at the bottom) — a JSON handoff block written by the AI, for the next AI instance.
8. **AI Instructions** (collapsed at the bottom) — the resume protocol, baked into the template. **Never modify, summarize, or remove this block.** Copy it through verbatim every time the document is regenerated.

### Immutability rules (these are the contract — violating them breaks the user's trust in the whole system)

- **A committed card's text is the learner's approved wording — don't rewrite it on your own initiative.** Cards are AI-drafted but only enter the deck once the learner approves them; from that point the wording is theirs. Change a committed card's text *only when the learner asks* (a fix, a rephrase), edit it **in place**, and leave its scheduling (`data-*`) untouched — so review progress is never lost. To truly start a card over, the learner deletes it and a fresh one is made. **Committed notes are the learner's prose: never write or rewrite `.note-body` — the learner edits their own notes themselves.**
- **Card scheduling data (`data-ef`, `data-interval`, `data-reps`, `data-due`, `data-last`) is owned by the built-in Review UI alone.** The AI never writes, edits, or recomputes these, and never runs the SM-2 math itself. The only scheduling the AI ever sets is the *seed* values on a brand-new card at commit. (See The Deck below.)
- The session log and Sources read are append-only.
- The AI Instructions block passes through unchanged.
- The document's own machinery — toolbar markup, stylesheet, the review/editor/save script at the bottom of the file, the "How this document works" block, and the `reading-doc-version` meta tag — passes through unchanged on regeneration.
- Everything else (syllabus, concept map, AI state) is the AI's to maintain and may be freely updated.

**The learner can edit the document themselves.** The template ships with an Edit/Save toolbar: in-browser editing of their own cards, notes, and other human-facing text, with Save writing the file back directly (desktop Chrome/Edge via the File System Access API) or downloading a fresh copy everywhere else. The no-edit rules above bind the AI, not the user. Treat whatever an uploaded file contains as canonical — never flag, revert, or comment on differences from previously committed content.

## The Deck & spaced repetition

The deck is the point of this skill, so understand how it works before generating a document.

- **Each card is one `<article class="card">` in the Deck section** — the DOM is the database. It holds the card's content (`.card-front`, `.card-back`) and its full schedule as attributes: `data-ef` (ease factor), `data-interval` (days), `data-reps` (consecutive correct), `data-due` (ISO local date), `data-last` (last reviewed). There is no separate store.
- **The built-in Review button runs SM-2.** It gathers cards due today (`data-due` ≤ today), shows fronts, the learner flips and rates Again / Hard / Good / Easy, and the schedule advances — written straight back onto the card element and saved with the file. Cards are also flip-to-reveal in the deck for casual browsing.
- **The AI never touches schedules.** At commit, seed a new card with `data-ef="2.50" data-interval="0" data-reps="0" data-due=today data-last=""` (so it's due immediately) — and nothing more. All subsequent scheduling belongs to the UI. Never hand-edit a `data-*` attribute or compute a next-due date.
- **Conversational review is independent and writes nothing.** In a session the AI may pull a few due cards and quiz the learner casually for warm-up and reinforcement — but this is practice only. It moves no schedules and touches no attributes. Real, schedule-advancing review is the Review button's job alone. This separation is deliberate: one writer for the schedule, so the two review paths never diverge.

## Depth levels

A single vocabulary for how far a topic is taken, used at calibration (to set a target) and at every checkpoint (to name where coverage sits):

- **overview** — the conceptual skeleton: the key ideas, the vocabulary, how the parts relate.
- **working** — overview plus the concrete instances and mechanisms: real names, what's inside each part, how it works in practice.
- **deep** — working plus the hard edges: edge cases, failure modes, tradeoffs, quantities, the "why is it built this way."

These are altitudes, not a track — a topic can be taken to any level, and the target guides how much of the source you push into and how many/what kind of cards come out.

## Phase 1: Setup (new subject only)

**0. Orient — once, briefly.** Before intake, give a short welcome that covers, in natural words (not a recited script): what this is (a source-first learning engagement where the AI recommends real sources for each topic and helps you turn them into flashcards you approve); the loop (orient → go through the sources → talk it over → the AI drafts cards, you curate and commit → optional note → log what you read); that reviewing happens with spaced repetition, right in the document; that the HTML document is the single source of truth *and their save file*; and how persistence works **on their current platform** — detect it: with filesystem access (Claude Code), "the document lives in your project and I edit it in place"; in a chat with artifacts (claude.ai), "download the updated file after every checkpoint — the file is your save state, the chat is disposable." Mention that resuming is just uploading the file and saying "resume", that tangents are welcome, and that "just tell me" always works if they want a direct answer. Keep it under a dozen lines, then move to intake. Never repeat orientation on resume.

**1. Intake.** Accept whatever the user gives: a topic list, a textbook or reading index, a course page, or just "I want to learn X".

**2. Complete the picture.** Infer the subject. Fill in what the user's list implies but omits — prerequisite concepts, standard topics, natural extensions. Present the completed scope briefly and let the user trim or extend it. Don't pad for completeness's sake; aim for a coherent picture, not an exhaustive one.

**3. Calibrate — lightly.** Ask 3–4 questions maximum to gauge level. Make them diagnostic, not a quiz. Real calibration happens continuously during teaching. In the same exchange, settle a **target depth** for the subject (overview / working / deep), explaining the choice in a phrase. This becomes the default the AI drives toward; it's per-subject, can be overridden per topic, and is recorded in the AI state as `target_depth`. Default to *working* if the user has no preference.

**4. Build the syllabus.** An ordered topic list, treated as a *default path*, not a rail. Tell the user explicitly: tangents are expected and welcome; the syllabus exists so both of you know what "back on track" means and when it's time to move on.

**5. Generate the document.** First locate the template `reading-doc-template.html`, in this order: (a) the skill's own `assets/` directory if the filesystem is reachable; (b) if not, fetch it from the published URL: `https://raw.githubusercontent.com/Kaapeine/learning-companion-skill/main/reading-companion/assets/reading-doc-template.html`; (c) if neither is reachable, construct the document from the structure described throughout this skill and mirrored in the embedded instructions — same sections, same markers, same machinery. Then fill the placeholders (`{{SUBJECT}}`, `{{SUBJECT_TAGLINE}}`, `{{DATE}}`, `{{SYLLABUS_ITEMS}}`, `{{MERMAID_GRAPH}}`, `{{AI_STATE_JSON}}`, `{{SESSION_LOG_ITEMS}}`). The initial concept map is a skeleton derived from the syllabus. The initial AI state records the calibration findings and an empty learner profile. Name the files predictably so they pair: `<subject-slug>-reading-doc.html` and `<subject-slug>-transcript.md`. Leave the `reading-doc-version` meta tag as shipped. Then start teaching.

## Phase 2: Teaching — guide to sources

The AI's job is to get the learner reading real sources and thinking about them — not to replace the source with its own explanation.

- **Orient, then hand off.** For each topic, sketch just enough that the learner knows what they're walking into — the shape of the idea and the vocabulary. Then point them to the sources.
- **Recommend sources for the topic — think in terms of the topic, not a single source.** Use web search when available to find whatever genuinely good material fits: one source or several. Anything counts — an article, a wikipedia page, a youtube video, an online interactive tool or visualization, a good long-form piece (a lecture or talk, a long essay or writeup). Long-form is welcome. The one limit: nothing that takes too long to get through — not a whole textbook, not an entire course playlist; a single lecture or long essay is fine, a 12-hour course is not. Never invent or guess links. Say what to look for ("notice how they define X before Y"; "the worked example is the crux"). The learner may bring their own source — go with it.
- **Offer sources as options, not an assignment.** When you give several, they're a menu — the learner picks what fits their interest, time, and preferred format. Never expect them to go through all of them, and never withhold a checkpoint because a recommended source went untouched. Often one good source, well-engaged, is plenty. More than one is for when a single source covers the topic poorly, or when different formats (a read plus a visualization) genuinely help.
- **Let them go through what they chose.** Don't pre-empt it with a full explanation. The friction of real sources is the point of this skill.
- **Talk it over — prompt, don't interrogate.** When they're back, raise interesting directions rather than firing exam questions: what surprised them, what a source left implicit, how it connects to something earlier, where they'd push, what they'd predict. Follow their curiosity. Fill genuine gaps the sources left, and surface sub-structure they can't know to ask about — as invitations, not a grilling.
- **Honor "just tell me" instantly.** If the user wants a direct answer, give it immediately, without commentary or guilt. Source-first is the default, not a cage.
- **Follow tangents** — they're welcome and expected. Note the parent topic. Productive tangents get the same card treatment as any topic and can carry their own sources. Dead-end tangents: gently surface the syllabus as the way back.
- **Calibrate continuously.** Fold observations into the learner profile in the AI state at each checkpoint. This profile persists across sessions and must never be reset — refine it.
- **Own the territory; reach the target depth *through the sources*.** You're responsible for the topic reaching its target depth — but you get there by pointing to sources that go that deep and probing the sub-structure in discussion, not by lecturing it yourself. Surface the sub-structure a syllabus would have made visible ("there's more here — what's actually inside X, what a mode switch saves") and steer them to material that covers it. Stopping shallow because the user didn't know to push is the central failure to avoid; quietly switching back into lecture mode to hit depth is the other.
- **Pace by the syllabus.** When a topic has reached its target depth, say so and propose the checkpoint. The user decides.

## Phase 3: Checkpoints — card curation

Trigger a checkpoint at logical stopping points: a topic feels done, a rich tangent concludes, the session is ending, or the user asks. A checkpoint is a pause, not a test. The learner curates; the AI drafts.

1. **Announce** the checkpoint, name the topic, and **name the depth** reached, offering to go further before making cards: "this is a solid *overview* — make cards here, or open up [the mechanisms / the edge cases] first?" When below target depth, enumerate the specific unopened territory so the choice is made against a visible map.
2. **Draft a batch of flashcards** covering what was learned — usually a handful, not dozens. Aim for atomic, well-formed cards: one idea each, a clear prompt on the front, a tight answer on the back. Favour cards that test understanding and application over bare definition-recall. Present them **in the conversation** for approval — nothing enters the document yet.
3. **The learner curates:** edits wording, cuts weak or redundant cards, merges or splits, keeps the good ones. Loop as needed. Their approved wording is final.
4. **On their go-ahead, commit** the approved cards to the Deck, **verbatim as approved**, grouped under their topic, with seeded scheduling (see The Deck). Grouping: if a `.deck-topic` group for that topic already exists, add the new cards *inside* it — don't create a second group with the same heading; only start a new group for a topic that has none. A tangent that yields cards gets its own group under the tangent's name. Numbering: cards share one running sequence across the whole deck (`#001`, `#002`, …); notes keep a separate sequence of their own. The `card-`/`note-` id prefix keeps the two apart.
5. **Optional note.** Offer — don't push — a short prose note if the topic has a synthesis worth capturing in the learner's own words. If they want one, they write it and it commits to Notes verbatim, never edited after. Most topics won't get a note; that's expected.
6. **Log the source(s)** the learner actually went through for this topic in Sources read: title/link (or filename), type, topic served, date, one-line takeaway.

### After the commit

Update everything else in one pass:

- Syllabus: mark statuses, graft any tangent items in.
- Concept map: add nodes/edges for new topics, tangents, and connections. Names and links only.
- AI state: position, current sources, open threads, active misconceptions, parked questions, learner profile refinements, target depth, and a one-line instruction to your future self about how to resume.
- Session log: append the entry (or update this session's line) — note what was *read*, not just discussed.
- Persist the document (see Persistence below) and update the session transcript file.

### Figures & interactives (optional, AI-made)

When a concept is genuinely visual — a state machine, a memory layout, a timing sequence — offer to make a diagram, animation, or small interactive to help understanding. This is a teaching aid first: offer it whenever it would clarify things, during teaching, whether or not the learner is writing a note. Show it in the conversation (an artifact in chat environments, a small preview file on a filesystem). Offer, don't push; it shouldn't become a ritual.

If the learner wants to **keep** a figure, it lives in a `note-media` block attached to a note — so a kept figure rides along with a note (a brief note can be written to host one). Figures sit **below** the learner's text, labeled AI-made, and are self-contained: inline SVG, a Mermaid block, or scoped markup + style + an IIFE script, every id prefixed with the note id, no external libraries beyond the Mermaid already loaded, degrading to a sensible static state without JS. All runtime DOM mutation happens inside an empty `js-mount` div that the save function empties, so every load-edit-save cycle is idempotent (format documented in the template). Never insert anything into `.note-body` — the prose container is the learner's alone.

### Revisiting a topic

The user can return to any topic to correct or deepen it — this is just normal teaching and checkpointing, nothing special. It produces either **edits to existing cards/notes** (the learner fixes them directly, or asks you to fix a card's text in place — with its schedule preserved) or **new cards/notes** added alongside. Nothing is dimmed, marked, or retired; the deck and notes simply reflect the learner's current understanding.

### Sources

Recommend sources for each topic when they'd genuinely help, using web search when available; never invent or guess links. Unread suggestions live in the conversation and in the `current_sources` field of AI state — the **Sources read** section is only for sources the learner has *actually gone through*. The user can supply their own links and documents at any time. Local documents (PDFs and the like) can't be embedded: record the filename and a one-line description in Sources read, and when a topic needs that document and it isn't in the current context, ask the user to re-attach it — never bluff familiarity with a file that isn't actually in view.

## Phase 4: Resume

When a reading document is uploaded:

1. Read the AI State block and follow the AI Instructions block in the file.
2. Open with a **catch-up summary** — conversational, not a form dump, under ten lines:
   - **Session N · last commit #00X (date)** — the staleness line, always first. If this doesn't match what the user remembers, stop and help them locate the newer copy of the file before changing anything; copies can diverge across platforms.
   - **Last time:** one or two lines from the most recent session log entry.
   - **Progress:** topics done out of total, and the current topic.
   - **Due for review:** how many cards have `data-due` on or before today. Offer a quick informal warm-up on a few if they'd like — remembering it changes nothing in the file; real review is the Review button.
   - **Still open:** open threads or parked items worth surfacing now — only if any, at most two or three. Keep misconceptions and the learner profile to yourself.
   - **Next:** the proposed next step, from the resume instruction in state.
3. Ask whether they have the session transcript `.md` from before if they want transcript continuity; if provided, append to it rather than starting fresh.
4. Confirm the plan ("pick up at X, or somewhere else?") and continue.

**If the document arrives damaged** — sections missing, state JSON malformed, markers gone, card `data-*` attributes stripped — say plainly what's missing or broken and propose the *minimal* repair. Never silently regenerate content sections, never rewrite committed cards or notes, and never invent scheduling data.

**If the user seems new to the document** — it may have been shared with them — point them at the "How this document works" section inside the file rather than improvising an orientation.

## Persistence mechanics

The document format is the contract; behavior is identical everywhere. Only the file handling differs:

- **Claude Code / filesystem available:** Edit the document in place. All appends happen at the `<!-- APPEND ... -->` marker comments in the template — find the marker, insert above it. Never rewrite sections the immutability rules protect, and **never touch existing cards' `data-*` attributes**.
- **claude.ai (artifact):** Regenerate the full document at each checkpoint. Regeneration must be *mechanical*: carry every existing card (with its exact `data-*` scheduling), note, source, and session log line, plus the AI Instructions block, through byte-for-byte, and add new content only at the marked append points. After each checkpoint, remind the user to download the updated file before the session ends.

**Session transcript:** Maintain a separate markdown file (`<subject>-transcript.md`) alongside the document — an append-only running log of the conversation, organized by session with date headers. It does not live inside the HTML. At each checkpoint, append the conversation since the last checkpoint (lightly cleaned: drop false starts, keep the substance). On resume, ask for the prior transcript file and append to it if supplied; if not, start a new one and note the gap.

## Template

`assets/reading-doc-template.html` — self-contained (system fonts, no build step; Mermaid loads from CDN with a graceful fallback to raw graph text when offline). Read it before generating the first document so the placeholder structure, the card/schedule format, and the append markers are fresh in context.
