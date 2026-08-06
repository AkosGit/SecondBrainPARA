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

The project index body auto-lists its board and its files grouped by topic. The project now also appears on `AREAS/Consulting.md` and under "Consulting" on [[Index]] — without you touching either file.

**Then add a board:** new note in the same folder → insert `TasksTemplate` → it becomes `Acme Analytics Audit Tasks.md` with Backlog / In Progress / Done and inherits `area/consulting`.

---

## Example 3 — Tasks, and when a task deserves a note

On the board, most cards are just text:

```
- [ ] Request GA4 access @{2026-08-03}
- [ ] Export current event taxonomy
```

But "Rewrite the event naming convention" is real work with substance. So you create a note for it — new note in the project folder → `ProjectFileTemplate` → *Type?* `Resource` → topics inherited — and then **put its link first in the card**:

```
- [ ] [[Event naming convention v2]] — draft and get sign-off @{2026-08-10}
```

**What that buys you:** on [[Index]], the Backlog and In Progress tables show *Event naming convention v2* as a clickable link instead of raw card text. Cards without links still show their text — linking is optional, use it when the task has thinking behind it.

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

Create a note → `IdeaTemplate` → *Area?* `Learning` → *Topics?* `analytics/attribution` → *Maturity?* `seed`.

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

## Example 6 — Developing ideas over time

A `seed` is a note you dropped and walked away from. The Index's **"Seeds needing development"** table keeps every `seed` and `developing` idea visible until you promote it.

Later you connect the attribution idea to something a client said, and rewrite it properly. Change one field by hand:

```yaml
status: developing   # → evergreen once it's genuinely settled
```

`evergreen` notes disappear from the seeds table — they're finished thinking, ready to be built on. Promoting is manual on purpose; nothing else can judge whether you actually understand something.

Two more nudges live on [[Index]]: the **serendipity block** shows three random idea notes every time you open the vault, and the spaced-repetition plugin is there if you want resurfacing to be deliberate rather than random.

---

## Example 7 — When a topic gets crowded

Six months in, `topic/analytics/attribution` has 12 notes. That's your signal — not before.

**Do:** create a note `Attribution — Map of Content`, tag it with the area it mostly serves, and paste query blocks filtered on that topic tag (copy the pattern from any `AREAS/` hub note). Now you have a curated entry point that maintains itself.

**Why wait for 12?** Maps of content built upfront are guesses about structure you don't have yet. Built from evidence, they describe what actually accumulated.

---

## Example 8 — Finishing a project

The Acme audit ships. **Do:** drag `PROJECTS/Acme Analytics Audit/` into `ARCHIVE/`. That's it.

The tags travel with the files, so the project drops off the active list and appears under "Archived projects" on [[Index]] automatically. Anything worth keeping beyond the engagement — the naming convention, the lessons — should already be an `Idea` or `Resource` note tagged by topic, so it stays findable after the project is cold.

This folder move is the **only** time the system asks you to file anything by hand.

---

## Cheat sheet

| I want to… | Do this |
|---|---|
| Save something to read | `Cmd/Ctrl+Shift+S`, or share to Obsidian from your phone |
| Start an area | New note in `AREAS/` → `AreaIndexTemplate` |
| Start a project | New folder in `PROJECTS/` → note → `ProjectIndexTemplate` → then `TasksTemplate` |
| Add a note to a project | Note in the project folder → `ProjectFileTemplate` (pick Resource/Idea/Source) |
| Write a permanent note | `IdeaTemplate` — area + topics + maturity |
| Add reference material | `ResourceTemplate` into `RESOURCES/` |
| Make a task clickable on the Index | Put its note's `[[wikilink]]` first in the kanban card |
| Mark reading done | Write the idea note, backlink both ways, `reading_status: integrated` |
| Promote an idea | Edit `status:` → `developing` → `evergreen` |
| Finish a project | Move its folder to `ARCHIVE/` |

## Five rules that keep it working

1. **One area per note, topics as many as fit.** The single area is what stops the vault becoming a tag soup where everything belongs everywhere.
2. **Never hand-write frontmatter.** Always a template. Hand-typed tags drift (`#analytics` vs `#Analytics`) and drifted tags are invisible to queries.
3. **Capture is dumb, classification is deliberate.** Tag on the way out (the idea), never on the way in (the source).
4. **The board owns task status. Frontmatter never does.**
5. **Reading is done when an idea note exists in your own words.** Not when you've highlighted it. Not when you've saved it.

The technical contract behind all of this: [[SYSTEM]]. Capture setup: [[Capture — Setup and Usage]].
