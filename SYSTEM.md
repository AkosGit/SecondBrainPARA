---
Parent: "[[Index|Link]]"
Type: Documentation
tags:
  - area/meta
---
# SecondBrainPARA — Technical System Reference (v2.1)

Technical description of the vault's architecture after the v2 metadata migration (2026-07-12), amended by the v2.1 changes of 2026-08-14 (§8). Audited against the vault on disk on 2026-08-14; discrepancies found are recorded in §9, not silently papered over. The vault still follows **PARA** (Projects, Areas, Resources, Archive) over plain Markdown, but classification moved from scalar frontmatter fields (`Area` / `Category` / `Subcategory`) to **namespaced tags**. Folders are structural scaffolding only; all intelligence lives in `Index.md`, `Boards.md`, the templates under `Templates/`, and the plugin configuration under `.obsidian/`.

A pre-migration snapshot of every note and template was saved outside the vault before the rewrite.

---

## 0. How it works — quick tour

**One idea: everything is a tagged note.** There are no category folders and no `Area`/`Category`/`Subcategory` fields anymore. Every note carries frontmatter `tags`, and two tag namespaces do all the classification work: `area/…` says *which part of your life this belongs to* (exactly one per note — e.g. `area/health`), and `topic/…` says *what it's about* (as many as you like, nested freely — e.g. `topic/programming/python`). Dashboards are just saved queries over these tags, so a note is "filed" the moment it's tagged, wherever it physically sits.

**Creating things.** You never write frontmatter by hand — templates do it. Each template prompts you with dropdowns built from the tags that already exist in the vault, so the vocabulary stays consistent:

1. **New area** → new note in `AREAS/` from `AreaIndexTemplate`. One flat note per area (e.g. `AREAS/Health.md`), no folder. This hub note automatically lists every project, idea, resource, and source tagged with its area.
2. **New project** → new folder in `PROJECTS/`, then a note from `ProjectIndexTemplate` (becomes `00000.md`). You pick the area (one) and topics (any). **The kanban board is created for you at the same time.** The index body auto-lists that board and all project files grouped by topic.
3. **Notes inside a project** → `ProjectFileTemplate`. You pick what it is (Resource / Idea / Source); it inherits the project's tags so it shows up everywhere the project does. A Resource also gets offered a board card, link only.
4. **Standalone notes** → `IdeaTemplate` (your own thinking, lands in `IDEAS/`) or `ResourceTemplate` (a finished artifact, lands in `RESOURCES/`) — both ask for area + topics. When an idea arrives too fast for four prompts, `Cmd/Ctrl+Shift+I` runs `QuickIdeaTemplate`: title, area, done — it lands as `status: seed` and the Index nags you until you develop it.
5. **Captured reading** → `SourceTemplate` into `Inbox/`. Sources are deliberately untagged at capture; tags are assigned later on the Idea note you extract from them.

**Tasks.** Each project gets one kanban board, created with the project. The board is the source of truth for status — a task's state *is* the list it sits in. If a task is big enough to deserve a note, put a wikilink to that note first in the card: `Index.md` then shows the note title as the clickable task entry. Cards without links just show their text. Due dates: `@{YYYY-MM-DD}`.

**The daily loop.** Open Obsidian → `Index.md` loads automatically. Top of the page: your In Progress and Backlog tasks across all projects, each a click away. Below that: projects grouped by area, your reading queue from the Inbox, your ideas, and all notes grouped by topic. When a project finishes, promote its finished artifacts to `RESOURCES/` and drag the folder to `ARCHIVE/` — the only filing the system ever asks of you.

Everything below is the precise contract behind this tour.

---

## 1. Metadata contract

### 1.1 Tag namespaces

All classification is carried by frontmatter `tags`. Two reserved namespaces:

| Namespace | Meaning | Cardinality | Example |
|---|---|---|---|
| `area/*` | PARA Area membership (ongoing responsibility). | **Exactly one** per project index / resource / idea. Drives grouping, hub queries, and accountability. | `area/health` |
| `topic/*` | Subject classification. Nested freely — replaces the old fixed Category→Subcategory hierarchy. | Zero or more. | `topic/programming/python` |

Anything outside these namespaces is a free ad-hoc tag with no system meaning.

Conventions: tags are lowercase, slugified (`[a-z0-9-]`, nested with `/`). Templates enforce this via a shared `slugify()` helper and suggester prompts that read the *existing* tag vocabulary from the vault — controlled vocabulary without fixed fields. Dataview queries match with `contains(file.tags, "#area/x")` (`file.tags` expands nested tags, so `#topic/programming` also matches notes tagged `#topic/programming/python`; `file.etags` holds only explicit tags and is used for display).

### 1.2 Type field

One `Type` value per note:

| Type | Meaning | Where |
|---|---|---|
| `Index` | Hub/dashboard note: the main `Index.md`, `Boards.md`, area hubs in `AREAS/`, project `00000.md` files, `Inbox/00000.md`. Distinguished by `Parent` and location, not by extra fields. | anywhere |
| `Resource` | **Finished artifact** — a piece of work that is done. Not a place to park reference material you haven't processed; that's a `Source`, and your thinking about it is an `Idea`. | `RESOURCES/` (standalone) or inside a project folder |
| `Idea` | Permanent note in your own words, at any stage of development; carries `status` and `source-notes: []` backlinks. Provenance is recorded by `source-notes`, **not** by location: an idea you had in the shower and an idea extracted from a Source are the same object and live in the same place. | `IDEAS/` (standalone) or inside a project folder |
| `Source` | Captured external material. **Must** carry a `source:` field (URL/book/person) — that is the crisp Source-vs-Resource line. Lands in `Inbox/` with `reading_status`. | `Inbox/`, later anywhere |
| `Tasks` | Kanban board file (`kanban-plugin: board`). | project folders |
| `Daily` | Daily/health log notes. Outside the PARA taxonomy; fixed `area/health` tag. ⚠️ **Not currently wired** — see §9. | daily notes |
| `Documentation` | Meta notes about the vault itself (this file, intake plans). | root |

### 1.3 Other fields

- `Parent` — upward link for hierarchy audit. Project files → project `00000`; project indexes / resources / ideas → their **area hub note**; hubs → `[[Index]]`.
- `Project` — folder name, stamped on project indexes, project files, and boards.
- `source`, `reading_status`, `ideas-extracted` — Source-note intake fields (unchanged from the v1 intake plan).
- `status` — Idea-note maturity: `seed` → `developing` → `evergreen`. Stamped `seed` at creation; you promote it by hand as the note matures. The Index surfaces every `seed`/`developing` idea until promoted. **`status` is what carries "this is a raw, undeveloped thought"** — there is no separate staging lane for ideas, and deliberately so: a second folder for half-formed ideas would duplicate what `seed` already says.
- **Removed fields:** `Area`, `Category`, `Subcategory`. Do not reintroduce them; every query now reads tags.

---

## 2. Folder topology

```
SecondBrainPARA/
├── Index.md              ← global dashboard (opens on startup via Homepage)
├── Boards.md             ← list of kanban boards (task tables moved to Index)
├── SYSTEM.md             ← this file
├── Capture — Setup and Usage.md          ← capture routes, per device
├── How to Use This Vault — Walkthrough.md ← worked examples
├── Information intake*.md ← v1 intake research and plans (Type: Documentation)
├── Inbox/                ← flat staging lane for Type: Source captures
│   └── 00000.md          ← inbox hub (Type: Index)
├── AREAS/                ← FLAT area hub notes: AREAS/Health.md, AREAS/Work.md …
├── PROJECTS/<Name>/      ← one folder per project
│   ├── 00000.md          ← project index (Type: Index, tags: [area/x, topic/…])
│   ├── <Name> Tasks.md   ← kanban board (Type: Tasks)
│   └── *.md              ← project files (Type: Resource | Idea | Source)
├── IDEAS/                ← standalone Idea notes, any maturity (tag-classified, flat)
├── RESOURCES/            ← standalone finished artifacts (tag-classified, flat)
├── ARCHIVE/              ← completed project folders, moved here wholesale
└── Templates/            ← 9 templates (§3)
```

`TASKS/` also exists at the root. It is a **v1 leftover with no role in v2** — boards live in project folders — and is empty. Safe to delete; nothing queries it.

Key change from v1: **AREAS/ contains flat hub notes, not folders.** An area exists iff a hub note exists. Area membership is a tag; the hub note is a saved set of Dataview queries over that tag (projects, ideas, resources, sources). Archiving is still folder-based: move the project folder to `ARCHIVE/` — the one place a folder carries meaning.

---

## 3. Templates

All templates follow one idiom: **gather answers first (prompts/suggesters), then emit the note — frontmatter included — as literal template output.** Frontmatter is *not* written with `processFrontMatter` at creation time: that call races Templater's own write of the rendered body and the frontmatter gets silently overwritten (the v2.0 bug, fixed 2026-07-27). `setTimeout`-deferred writes are the same race, merely one it usually wins; both patterns are banned in creation templates. Use `processFrontMatter` only for editing notes that already exist. Shared helpers (inlined per template): `slugify()`, `allFrontmatterTags(prefix)` (scans vault for existing `area/*` / `topic/*` vocabulary), `pickAreaHub()` (suggester over flat notes in `AREAS/`, with "new area" fallback), `pickTopicTags()` (multi-select loop with "New topic…" / "Done").

| Template | Creates | Behaviour |
|---|---|---|
| `AreaIndexTemplate` | Area **hub** note, flat in `AREAS/` | Prompts area name → tag `area/<slug>`; body queries projects/ideas/resources/sources by that tag. |
| `ProjectIndexTemplate` | `00000.md` **and the project's board** | Pick one area hub (Parent + `area/*` tag) + optional topics; body lists the board, groups project files by `topic/*` (dataviewjs), plus a flat table. Then writes `<Name> Tasks.md` via `vault.create`, inheriting the area tag just chosen. No-ops if the folder already holds a `Type: Tasks` note or that filename is taken; a failed create emits a Notice and never blocks the index. |
| `ProjectFileTemplate` | Note inside a project folder | Prompts `Type` (Resource/Idea/Source), **inherits** the project index's tags, optionally adds topics; Parent → project `00000`. For `Type: Resource` it then offers *Backlog / In Progress / No card* and appends a link-only card to the project's board (see §4). Idea notes additionally get the source picker and reciprocal backlinks. |
| `ResourceTemplate` | Finished artifact in `RESOURCES/` | Pick area hub + topics; `Type: Resource`; Parent → hub. |
| `IdeaTemplate` | Idea note | Pick area hub + topics + maturity (`status: seed` default) + optional source notes; adds `source-notes`; Parent → hub. **Moves the note to `IDEAS/`** unless it was created under `PROJECTS/`, `ARCHIVE/` or `IDEAS/` already. Body ends with a live **Related notes** footer (dataviewjs: every note sharing a `topic/*` tag). |
| `QuickIdeaTemplate` | Idea note, fast path | Two prompts (title, area) → `Type: Idea`, `status: seed`, `source-notes: []`, one `area/*` tag. Topics, maturity and source links deferred to when you develop the note. Bound to `Cmd/Ctrl+Shift+I` via QuickAdd, target folder `IDEAS/`. |
| `SourceTemplate` | Capture in `Inbox/` | **Share-aware.** Guard skips notes already stamped `Type: Source` (Web Clipper); extracts the first URL from the body if present (Android share) instead of prompting; uses the filename as title unless "Untitled". The entire note is emitted through `tR`, so the guard branch can hand the file back byte-for-byte instead of letting Templater overwrite a clipper capture. Also auto-applied by Templater's folder template to any new file in `Inbox/`. Tags deferred to the Idea note. |
| `TasksTemplate` | Kanban board in a project folder | **Manual path only** — new projects get their board from `ProjectIndexTemplate`; use this to add one to a project that never got it. Inherits the project's `area/*` tag from `00000.md`; `Type: Tasks`, `kanban-plugin: board`, Backlog / In Progress / Done lists. ⚠️ Produces the same skeleton as `createBoard()` in `ProjectIndexTemplate` — **the two must be kept in sync.** |
| `DailyNoteTemplate` | Daily health log | `Type: Daily`, `tags: [area/health]`; Modal Forms `master_health_log` unchanged. ⚠️ Insert by hand — periodic-notes is unconfigured, so nothing applies this automatically (§9). |

---

## 4. Task convention

Kanban cards remain the **source of truth for status** (which list the card sits in). Convention, not requirement:

- If a card contains a wikilink, the **first link** is the note that *represents* that task; the Index renders it as the clickable task title.
- Cards without links fall back to the card text as title.
- Due dates use the Kanban plugin's `@{YYYY-MM-DD}` syntax, parsed by regex in the Index queries.

**Card creation from the note side.** `ProjectFileTemplate` writes the card for you when the note is a `Type: Resource` inside a project: it locates the project's board (the file in the same folder with `Type: Tasks`), asks *Backlog / In Progress / No card*, and appends `- [ ] [[path|Title]]` — link only, no due date, no status — to the end of that list. Implementation notes: the insertion walks to the end of the target list and backs up over trailing blank lines so the card lands under the last existing card rather than in the gap; it stops at the next `##` heading or the `%% kanban:settings` block, which it never touches. If the project has no board, or the named list is missing, it emits a Notice and skips rather than failing the note creation. Edit the card afterwards to add a due date or prose — just keep the wikilink first.

`Index.md` renders per-list tables (In Progress, Backlog) by flattening `file.tasks` across `PROJECTS/` and reading `meta(Tasks.section).subpath`; the title column is `choice(length(Tasks.outlinks) > 0, Tasks.outlinks[0], split(Tasks.text, " @")[0])`. `Boards.md` only lists the boards themselves.

---

## 5. Index.md layout

Top-to-bottom: **Tasks** (In Progress, Backlog) → **Projects by area** (dataviewjs grouped on the `area/*` namespace; untagged projects fall into `unsorted`) → **Areas** (hub notes) → **Reading queue** (WIP counter that flips to a warning above 10 undigested sources; inbox/reading with age; **stale list** for sources older than 14 days, oldest first; integrated) → **Ideas** (seeds/developing needing work; a full table of every Idea note with its maturity and folder; plus a serendipity block that surfaces 3 random Idea notes on every open) → **Notes by topic** (dataviewjs grouped on `topic/*`; replaces the old Category→Subcategory report; multi-topic notes appear once per topic by design) → **Resources** (finished artifacts) → **Archived projects** → **Audit** (notes without `Parent`, project indexes missing an `area/*` tag, Idea notes missing an `area/*` tag).

The Ideas queries run `FROM ""` — vault-wide — so an Idea note is surfaced wherever it sits. `IDEAS/` is an ergonomic default, not a requirement the dashboards enforce.

---

## 6. Plugin stack

Unchanged from v1 — see `.obsidian/community-plugins.json` (17 enabled, verified 2026-08-14). Load-bearing: **Templater** (template JS; templates folder `Templates`, `trigger_on_file_creation: true`, one folder mapping `Inbox` → `SourceTemplate`), **Dataview/DataviewJS** (all dashboards), **Kanban** (boards), **QuickAdd** (the two capture commands), **Homepage** (opens `Index.md` on startup, confirmed `openOnStartup: true`). Supporting: modalforms (`master_health_log` form present), obsidian-git, system3-relay, linter, outliner, various-complements, find-unlinked-files, spaced-repetition, paste-image-rename, better-export-pdf, pdf-plus.

**periodic-notes is enabled but has an empty config**, and the core Daily Notes plugin has none either — so no daily-note folder, date format or template is set. Nothing in the system depends on this; it only means `Type: Daily` is currently aspirational (§9).

---

## 7. Workflows

Capture setup and per-device usage (QuickAdd, phone share sheet, Web Clipper): see [[Capture — Setup and Usage]].

Worked examples of every workflow below: see [[How to Use This Vault — Walkthrough]].

- **New area:** create note in `AREAS/` from `AreaIndexTemplate`. That's the whole operation — no folder.
- **New project:** create folder in `PROJECTS/`, add a note from `ProjectIndexTemplate`. That is the whole operation — the note becomes `00000.md` and the board is created beside it in the same pass. `TasksTemplate` is only for retrofitting a board onto a project that predates this.
- **Capture a source (in-vault):** press `Cmd/Ctrl+Shift+S` — the QuickAdd "New Source" command creates a stamped Source note in `Inbox/` and prompts for title + URL. The Web Clipper covers capture from the browser.
- **Capture an idea:** press `Cmd/Ctrl+Shift+I` — QuickAdd "New Idea" runs `QuickIdeaTemplate` and drops a `status: seed` note into `IDEAS/` after two prompts. This is the path for a thought that didn't come from anything you read. An idea you *did* extract from a Source goes to the same place — write it with `IdeaTemplate` so the source picker fills `source-notes` here and `ideas-extracted` on the Source.
- **Capture (phone, share sheet):** new notes default to `Inbox/`, and a Templater folder template auto-runs `SourceTemplate` on any new file there. Share a page → Obsidian creates the note → the template detects the URL in the body and the title in the filename and stamps the full Source frontmatter. A guard skips notes that arrive already stamped (Web Clipper). Caveat: Templater may skip auto-applying to files created with content — if a shared note comes through raw, insert `SourceTemplate` manually; the same detection logic runs.
- **Capture → integrate (intake v1, unchanged):** clip/capture → `Inbox/` `Type: Source` → read & highlight → write `Type: Idea` note(s) in your own words (tags assigned here), backlink both ways, flip `reading_status: integrated`.
- **New task with a note:** add a card to the project board; if the task deserves a note, create the note (any type) and put its wikilink first in the card.
- **Develop ideas:** new Idea notes start as `status: seed` and sit in the Index's "Seeds needing development" table until promoted to `developing`, then `evergreen` (edit the field by hand). The related-notes footer on each Idea note and the serendipity block on the Index exist to trigger collisions between ideas.
- **Promote a topic to a hub (rule of thumb, not machinery):** when a `topic/*` tag accumulates ~10+ notes, create a hub note for it — same pattern as an area hub, with queries filtered on that topic tag. Maps of Content emerge from evidence; don't create them upfront.
- **Inbox discipline:** the reading queue shows a hard number against a cap of 10. If the warning is showing, digest or delete before capturing more. Anything older than 14 days appears in the stale list — integrate it or admit you never will and delete it.
- **Archive a project:** first promote its **finished artifacts** — anything the project actually produced — out to `RESOURCES/`, then move the folder to `ARCHIVE/`. `ARCHIVE/` is for the record of how the work happened; `RESOURCES/` is for the work itself, which stays live and findable after the project is dead. Ideas the project generated that outlive it get promoted to `IDEAS/` the same way. Tags travel with the files; the Index picks all of it up automatically.

## 8. Change log

### v2.1 — Ideas get a home, Resources get a definition (2026-08-14)

Two gaps closed. **Ideas had no standalone location** — `RESOURCES/` was for Resources, `Inbox/` is Source-only (its Templater folder template stamps anything created there as `Type: Source`), so in practice Idea notes only ever appeared inside project folders. Added flat `IDEAS/`; `IdeaTemplate` now moves standalone ideas there, project-scoped ideas stay put.

**`Resource` was defined as "your own evergreen reference material,"** which overlapped with `Idea` and gave unfinished work somewhere to hide. Resource now means *finished artifact*. The clean split: `Source` = someone else's material, `Idea` = your thinking at any maturity, `Resource` = a thing you finished.

Two designs were considered and rejected:

- **`Inbox/Sparks/` as a staging lane for raw ideas.** Rejected as redundant with `status: seed`, which already means "undeveloped." It would also have inherited the `Inbox` Templater folder mapping (folder templates match by path prefix) and stamped every spark `Type: Source`, and its parentless notes would have shown up in the Index's `FROM "Inbox"` Parent audit.
- **Filing source-derived ideas separately from original ones.** Rejected because provenance is already recorded by `source-notes` / `ideas-extracted`, and it stops mattering once the note exists. Two folders for one type just means not remembering which one you used.

Added: `IDEAS/`, `QuickIdeaTemplate` + the "New Idea" QuickAdd command on `Cmd/Ctrl+Shift+I`, an all-ideas table and an ideas-without-an-area audit on `Index.md`, and the artifact-promotion step when archiving a project.

Also added, same day: **`ProjectIndexTemplate` now creates the project's board itself**, so starting a project is one folder and one note. It builds the board inline via `vault.create` rather than invoking `TasksTemplate`, because that template inherits the area tag by reading `00000.md` out of the metadata cache — and at that moment the index hasn't been written, so the cache is empty and the board would land with `tags: []`. Same class of race as the v2.0 frontmatter bug. The cost is that the board skeleton now exists in two files; both carry a ⚠️ sync comment pointing at the other, and they render byte-identical today.

Also added, same day: `ProjectFileTemplate` now offers to append a link-only kanban card when the note is a `Type: Resource` inside a project (§4). Scoped to Resources because a Resource in a project is a deliverable that usually owes a task, while Ideas and Sources usually don't. The card carries no metadata — the board still owns status, and the card is a pointer to the note, not a duplicate of it.

### v1 → v2 (2026-07-12)

The PARA folders were empty at migration time, so no note frontmatter needed conversion. Changed in one pass: all 8 templates, `Index.md`, `Boards.md`, `Inbox/00000.md`, this file. The v1 `Area`/`Category`/`Subcategory` contract, the AREAS folder-per-area layout, and the old grouped Category report are retired. The intake-plan documents (`Information Intake — v1 *`) still describe the old `Area` fields in places; their Source/Idea pipeline is untouched and remains valid.

---

## 9. Known gaps and housekeeping

State of the vault as audited 2026-08-14. None of these break anything; they're the difference between what this document describes and what is on disk.

| Item | Status | What to do |
|---|---|---|
| `TASKS/` at the root | v1 leftover, empty | Delete it. Boards live in project folders; nothing queries this path. |
| `orphaned files output.md` | Scratch output from find-unlinked-files, no frontmatter, contains `jjjjjj` | Delete it. |
| `Information intake.md` (77 KB) | No frontmatter, so invisible to every dashboard | Stamp `Type: Documentation` + `tags: [area/meta]` like the other two intake docs, or archive it. |
| periodic-notes / Daily Notes | Both unconfigured; zero `Type: Daily` notes exist | Configure a daily folder + `DailyNoteTemplate`, or drop `Type: Daily` from §1.2 and delete `DailyNoteTemplate`. Don't leave it half-declared. |
| `PROJECTS/Fatigue/` | Predates automatic board creation, has no board | Add one via `TasksTemplate` if the project is live. |
| Test notes | `test file.md`, `test idea.md`, `test resource.md`, `water.md` across `Inbox/`, `RESOURCES/`, `PROJECTS/` | Scaffolding from building the system. Delete when you start using it for real. |

**Verified accurate** at the same audit: the folder topology in §2, all 9 templates in §3 exist and match their descriptions, the `Index.md` section order in §5, the Templater and QuickAdd configuration in §6, and the `Type` values in §1.2 — live frontmatter contains only `Index` (6), `Documentation` (5), `Resource` (3), `Source` (2), `Idea` (2), `Tasks` (1), all declared.
