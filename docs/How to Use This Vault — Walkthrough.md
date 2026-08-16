---
Parent: "[[Index|Link]]"
Type: Documentation
tags:
  - area/meta
---
# How to Use This Vault — Walkthrough

Learn the system by watching it get used. One running story — setting up a work area, running a client project, reading a paper, developing an idea — with the exact keystrokes and the exact frontmatter that results. Read once end to end; after that use the cheat sheet at the bottom.

The whole system rests on one rule: **notes are classified by tags, not by where they sit.** `area/…` says which part of your life a note belongs to (exactly one). `topic/…` says what it's about (as many as you want, nested). Every dashboard is a query over those tags. You never type frontmatter by hand — templates do it.

---

## Example 1 — Setting up an area

You want to keep client consulting work separate from personal learning. That's two areas.

**Do:** create a new note directly inside `AREAS/`, insert `AreaIndexTemplate`, answer the prompt with `Consulting`.

**You get** `AREAS/Consulting.md`:

```yaml
---
Parent: "[[Index|Link]]"
Type: Index
tags:
  - area/consulting
---
```

The note body is already a dashboard — it lists every project, idea, resource, and source tagged `#area/consulting`, all empty for now. Repeat for `Learning`.

**Why it matters:** an area is one note, not a folder. Nothing needs filing into it — tagging is filing.

> Rule of thumb: 5–9 areas total. If you have 20, they're topics, not areas.

---

## Example 2 — Starting a project

A client hires you to audit their analytics setup.

**Do:** create folder `PROJECTS/Acme Analytics Audit/` → new note inside it → insert `ProjectIndexTemplate` → it renames itself `00000.md` and asks:
- *Area?* → pick `Consulting` (one only — this is the accountability link)
- *Topics?* → `New topic...` → type `analytics/tracking` → then `New topic...` → `client/acme` → then `Done`

**You get** `PROJECTS/Acme Analytics Audit/00000.md`:

```yaml
---
Parent: "[[AREAS/Consulting|Link]]"
Type: Index
Project: Acme Analytics Audit
tags:
  - area/consulting
  - topic/analytics/tracking
  - topic/client/acme
---
```

**And you get the board, free.** `Acme Analytics Audit Tasks.md` is created in the same folder at the same moment — Backlog / In Progress / Done, `kanban-plugin: board`, inheriting `area/consulting`. You don't insert `TasksTemplate` yourself; that's now only for adding a board to an older project that never got one. If a board already exists in the folder, nothing is overwritten.

The project index body auto-lists that board and the project's files grouped by topic. The project also appears on `AREAS/Consulting.md` and under "Consulting" on [[Index]] — without you touching either file.

So a project is **one folder and one note**. Everything else assembles itself.

---

## Example 3 — Tasks, and when a task deserves a note

On the board, most cards are just text:

```
- [ ] Request GA4 access @{2026-08-03}
- [ ] Export current event taxonomy
```

But "Rewrite the event naming convention" is real work with substance. So you create a note for it — new note in the project folder → `ProjectFileTemplate` → *Type?* `Resource` → topics inherited → *Add a card to the board?* `Backlog`.

That last prompt writes the card for you, link only:

```
- [ ] [[PROJECTS/Acme Analytics Audit/Event naming convention v2|Event naming convention v2]]
```

It only appears for `Resource` notes — a Resource inside a project is a deliverable, so it usually owes a task. Pick *No card* when it doesn't. Ideas and Sources never prompt; add those cards by hand if you want them.

**Adding detail by hand.** The card is yours to edit afterwards — a due date, a note on what "done" means:

```
- [ ] [[Event naming convention v2]] — draft and get sign-off @{2026-08-10}
```

**What that buys you:** on [[Index]], the Backlog and In Progress tables show *Event naming convention v2* as a clickable link instead of raw card text. The **first** link in a card is the one that wins, so keep it first when you add prose. Cards without links still show their text — linking is optional, use it when the task has thinking behind it.

**Status lives on the board, never in frontmatter.** A task is "in progress" because its card sits in the In Progress list. Drag the card, and the Index updates. Don't add a `status` field to task notes — that's two sources of truth.

---

## Example 4 — Capturing while reading (the ingestion loop)

You're on your phone and someone links a paper on marketing attribution models.

**Do:** Share → Obsidian (or the Web Clipper). Nothing else. No title cleanup, no tagging, no deciding where it goes.

**You get**, in `Inbox/`:

```yaml
---
Type: Source
Parent: "[[Inbox/00000|Link]]"
reading_status: inbox
source: https://example.org/attribution-models
author: ""
ideas-extracted: []
tags: []
---
```

Note `tags: []` — **deliberately empty.** Classifying something you haven't read yet is guessing. Classification happens later, on the idea you extract.

The note now appears in the **Reading queue** on [[Index]] with its age ticking up. Two pressure valves live there: a counter that turns into a warning above 10 undigested sources, and a "Stale (older than 14 days)" table.

> If the warning is showing, you may not capture more until you digest or delete. Deleting is a legitimate outcome — a source you've ignored for a month was never important.

Setup details for all three capture routes: [[Capture — Setup and Usage]].

---

## Example 5 — Digesting: source → idea

You read the paper and highlight as you go, into its `## Highlights` section. Highlighting is **not** the work. The work is the idea note.

**Do:** cluster the highlights by concept (not by the paper's chapter order). One cluster stands out: attribution models fail because they assume linear journeys. That's one idea → one note.

Create a note → `IdeaTemplate` → *Area?* `Learning` → *Topics?* `analytics/attribution` → *Maturity?* `seed` → *Source note(s)?* `Multi-Touch Attribution Models`. The note lands in `IDEAS/`, and picking the source there fills `source-notes` on the idea and `ideas-extracted` on the paper in one pass.

**You get:**

```yaml
---
Parent: "[[AREAS/Learning|Link]]"
Type: Idea
status: seed
source-notes: []
tags:
  - area/learning
  - topic/analytics/attribution
---
```

Now write the idea **in your own words** — if you can't explain it without quoting, you haven't understood it yet. Then close the loop, both directions:

1. In the idea note: `source-notes: ["[[Multi-Touch Attribution Models]]"]`
2. In the source note: `ideas-extracted: ["[[Attribution assumes journeys are linear]]"]` and `reading_status: integrated`

The source leaves the queue. **That's the only definition of "done" for reading in this system:** an idea note exists, in your words, that links back.

Scroll to the bottom of your new idea note — the **Related notes** footer already lists everything else tagged `topic/analytics/…`. That's where the Acme project's tracking notes show up next to a paper you read for fun, which is exactly the collision you're farming.

---

## Example 6 — An idea that came from nowhere

Walking to lunch, it occurs to you that clients never ask for attribution — they ask "which channel do I cut". Nothing prompted it. There's no source.

**Do:** press `Cmd/Ctrl+Shift+I` → *Idea?* `Clients ask what to cut, not what works` → *Area?* `Consulting`. Done — two prompts, no topics, no maturity question.

**You get,** in `IDEAS/`:

```yaml
---
Parent: "[[AREAS/Consulting|Link]]"
Type: Idea
status: seed
source-notes: []
tags:
  - area/consulting
---
```

`source-notes: []` is the whole record of "this came from my head" — and it's the *only* difference from the note in Example 5. Both are `Type: Idea`, both live in `IDEAS/`, both show up in the same tables. **Where an idea came from is frontmatter, not a folder.** Filing sourceless ideas somewhere separate would mean two homes for one type, and no memory of which one you used.

Nothing else is asked of you at capture, because a thought you're chasing down mid-thought is a thought you lose. The Index will keep it in "Seeds needing development" until you come back and add topics, which is where classification belongs anyway.

---

## Example 7 — Developing ideas over time

A `seed` is a note you dropped and walked away from. The Index's **"Seeds needing development"** table keeps every `seed` and `developing` idea visible until you promote it.

Later you connect the attribution idea to something a client said, and rewrite it properly. Change one field by hand:

```yaml
status: developing   # → evergreen once it's genuinely settled
```

`evergreen` notes disappear from the seeds table — they're finished thinking, ready to be built on. Promoting is manual on purpose; nothing else can judge whether you actually understand something.

Two more nudges live on [[Index]]: the **serendipity block** shows three random idea notes every time you open the vault, and the spaced-repetition plugin is there if you want resurfacing to be deliberate rather than random.

---

## Example 8 — When a topic gets crowded

Six months in, `topic/analytics/attribution` has 12 notes. That's your signal — not before.

**Do:** create a note `Attribution — Map of Content`, tag it with the area it mostly serves, and paste query blocks filtered on that topic tag (copy the pattern from any `AREAS/` hub note). Now you have a curated entry point that maintains itself.

**Why wait for 12?** Maps of content built upfront are guesses about structure you don't have yet. Built from evidence, they describe what actually accumulated.

---

## Example 9 — Finishing a project

The Acme audit ships. Two moves, in this order.

**First, promote what the project produced.** The event naming convention you wrote is a **finished artifact** — that's what `Type: Resource` means. Move it to `RESOURCES/`. The lesson you drew about clients asking what to cut is thinking, not an artifact — move it to `IDEAS/`. Everything else (working notes, the half-finished draft, the meeting scraps) stays put.

**Then drag `PROJECTS/Acme Analytics Audit/` into `ARCHIVE/`.**

The split is about what you'll want later: `ARCHIVE/` is the record of *how the work happened*, and you'll rarely open it. `RESOURCES/` and `IDEAS/` are the things that outlive the engagement and stay live. Tags travel with the files either way, so the project drops off the active list and appears under "Archived projects" on [[Index]] automatically.

These two folder moves are the **only** time the system asks you to file anything by hand.

---

## Cheat sheet

| I want to… | Do this |
|---|---|
| Save something to read | `Cmd/Ctrl+Shift+S`, or share to Obsidian from your phone |
| Catch an idea before it's gone | `Cmd/Ctrl+Shift+I` — title + area, lands in `IDEAS/` as `status: seed` |
| Start an area | New note in `AREAS/` → `AreaIndexTemplate` |
| Start a project | New folder in `PROJECTS/` → note → `ProjectIndexTemplate` (the board is created for you) |
| Add a board to an old project | Note in the project folder → `TasksTemplate` |
| Add a note to a project | Note in the project folder → `ProjectFileTemplate` (pick Resource/Idea/Source; a Resource also offers a board card) |
| Write an idea properly | `IdeaTemplate` — area + topics + maturity + source notes; lands in `IDEAS/` |
| File something you finished | `ResourceTemplate` into `RESOURCES/` |
| Make a task clickable on the Index | Put its note's `[[wikilink]]` first in the kanban card |
| Mark reading done | Write the idea note, backlink both ways, `reading_status: integrated` |
| Promote an idea | Edit `status:` → `developing` → `evergreen` |
| Finish a project | Promote its artifacts to `RESOURCES/`, then move the folder to `ARCHIVE/` |

## Six rules that keep it working

1. **One area per note, topics as many as fit.** The single area is what stops the vault becoming a tag soup where everything belongs everywhere.
2. **Never hand-write frontmatter.** Always a template. Hand-typed tags drift (`#analytics` vs `#Analytics`) and drifted tags are invisible to queries.
3. **Capture is dumb, classification is deliberate.** Tag on the way out (the idea), never on the way in (the source).
4. **The board owns task status. Frontmatter never does.**
5. **Reading is done when an idea note exists in your own words.** Not when you've highlighted it. Not when you've saved it.
6. **A Resource is something you finished.** If it's still moving, it's an `Idea` with a `status`. Keeping unfinished work out of `RESOURCES/` is what stops it becoming a graveyard.

The technical contract behind all of this: [[SYSTEM]]. Capture setup: [[Capture — Setup and Usage]].
