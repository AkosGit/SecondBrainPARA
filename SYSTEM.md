---
Parent: "[[Index|Link]]"
Type: Documentation
tags:
  - area/meta
---
# SecondBrainPARA — Technical System Reference (v2)

Technical description of the vault's architecture after the v2 metadata migration (2026-07-12). The vault still follows **PARA** (Projects, Areas, Resources, Archive) over plain Markdown, but classification moved from scalar frontmatter fields (`Area` / `Category` / `Subcategory`) to **namespaced tags**. Folders are structural scaffolding only; all intelligence lives in `Index.md`, `Boards.md`, the templates under `Templates/`, and the plugin configuration under `.obsidian/`.

A pre-migration snapshot of every note and template was saved outside the vault before the rewrite.

---

## 0. How it works — quick tour

**One idea: everything is a tagged note.** There are no category folders and no `Area`/`Category`/`Subcategory` fields anymore. Every note carries frontmatter `tags`, and two tag namespaces do all the classification work: `area/…` says *which part of your life this belongs to* (exactly one per note — e.g. `area/health`), and `topic/…` says *what it's about* (as many as you like, nested freely — e.g. `topic/programming/python`). Dashboards are just saved queries over these tags, so a note is "filed" the moment it's tagged, wherever it physically sits.

**Creating things.** You never write frontmatter by hand — templates do it. Each template prompts you with dropdowns built from the tags that already exist in the vault, so the vocabulary stays consistent:

1. **New area** → new note in `AREAS/` from `AreaIndexTemplate`. One flat note per area (e.g. `AREAS/Health.md`), no folder. This hub note automatically lists every project, idea, resource, and source tagged with its area.
2. **New project** → new folder in `PROJECTS/`, then a note from `ProjectIndexTemplate` (becomes `00000.md`). You pick the area (one) and topics (any). Its body auto-lists the project's kanban board and all project files grouped by topic.
3. **Notes inside a project** → `ProjectFileTemplate`. You pick what it is (Resource / Idea / Source); it inherits the project's tags so it shows up everywhere the project does.
4. **Standalone notes** → `ResourceTemplate` (reference material you wrote) or `IdeaTemplate` (permanent notes in your own words) — both ask for area + topics.
5. **Captured reading** → `SourceTemplate` into `Inbox/`. Sources are deliberately untagged at capture; tags are assigned later on the Idea note you extract from them.

**Tasks.** Each project gets one kanban board (`TasksTemplate`). The board is the source of truth for status — a task's state *is* the list it sits in. If a task is big enough to deserve a note, put a wikilink to that note first in the card: `Index.md` then shows the note title as the clickable task entry. Cards without links just show their text. Due dates: `@{YYYY-MM-DD}`.

**The daily loop.** Open Obsidian → `Index.md` loads automatically. Top of the page: your In Progress and Backlog tasks across all projects, each a click away. Below that: projects grouped by area, your reading queue from the Inbox, and all notes grouped by topic. When a project finishes, drag its folder to `ARCHIVE/` — that's the only folder move the system ever asks of you.

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
| `Resource` | Your own evergreen reference material. | `RESOURCES/` or inside a project folder |
| `Idea` | Permanent note in your own words; carries `source-notes: []` backlinks. | anywhere |
| `Source` | Captured external material. **Must** carry a `source:` field (URL/book/person) — that is the crisp Source-vs-Resource line. Lands in `Inbox/` with `reading_status`. | `Inbox/`, later anywhere |
| `Tasks` | Kanban board file (`kanban-plugin: board`). | project folders |
| `Daily` | Daily/health log notes. Outside the PARA taxonomy; fixed `area/health` tag. | daily notes |
| `Documentation` | Meta notes about the vault itself (this file, intake plans). | root |

### 1.3 Other fields

- `Parent` — upward link for hierarchy audit. Project files → project `00000`; project indexes / resources / ideas → their **area hub note**; hubs → `[[Index]]`.
- `Project` — folder name, stamped on project indexes, project files, and boards.
- `source`, `reading_status`, `ideas-extracted` — Source-note intake fields (unchanged from the v1 intake plan).
- `status` — Idea-note maturity: `seed` → `developing` → `evergreen`. Stamped `seed` at creation; you promote it by hand as the note matures. The Index surfaces every `seed`/`developing` idea until promoted.
- **Removed fields:** `Area`, `Category`, `Subcategory`. Do not reintroduce them; every query now reads tags.

---

## 2. Folder topology

```
SecondBrainPARA/
├── Index.md              ← global dashboard (opens on startup via Homepage)
├── Boards.md             ← list of kanban boards (task tables moved to Index)
├── SYSTEM.md             ← this file
├── Inbox/                ← flat staging lane for Type: Source captures
│   └── 00000.md          ← inbox hub (Type: Index)
├── AREAS/                ← FLAT area hub notes: AREAS/Health.md, AREAS/Work.md …
├── PROJECTS/<Name>/      ← one folder per project
│   ├── 00000.md          ← project index (Type: Index, tags: [area/x, topic/…])
│   ├── <Name> Tasks.md   ← kanban board (Type: Tasks)
│   └── *.md              ← project files (Type: Resource | Idea | Source)
├── RESOURCES/            ← standalone reference notes (tag-classified, flat)
├── ARCHIVE/              ← completed project folders, moved here wholesale
└── Templates/
```

Key change from v1: **AREAS/ contains flat hub notes, not folders.** An area exists iff a hub note exists. Area membership is a tag; the hub note is a saved set of Dataview queries over that tag (projects, ideas, resources, sources). Archiving is still folder-based: move the project folder to `ARCHIVE/` — the one place a folder carries meaning.

---

## 3. Templates

All templates keep the v1 idiom: prompt → rename → write frontmatter via `processFrontMatter`. Shared helpers (inlined per template): `slugify()`, `allFrontmatterTags(prefix)` (scans vault for existing `area/*` / `topic/*` vocabulary), `pickAreaHub()` (suggester over flat notes in `AREAS/`, with "new area" fallback), `pickTopicTags()` (multi-select loop with "New topic…" / "Done").

| Template | Creates | Behaviour |
|---|---|---|
| `AreaIndexTemplate` | Area **hub** note, flat in `AREAS/` | Prompts area name → tag `area/<slug>`; body queries projects/ideas/resources/sources by that tag. |
| `ProjectIndexTemplate` | `00000.md` in a project folder | Pick one area hub (Parent + `area/*` tag) + optional topics; body lists the board, groups project files by `topic/*` (dataviewjs), plus a flat table. |
| `ProjectFileTemplate` | Note inside a project folder | Prompts `Type` (Resource/Idea/Source), **inherits** the project index's tags, optionally adds topics; Parent → project `00000`. |
| `ResourceTemplate` | Note in `RESOURCES/` | Pick area hub + topics; `Type: Resource`; Parent → hub. |
| `IdeaTemplate` | Permanent note | Pick area hub + topics + maturity (`status: seed` default); adds `source-notes: []`; Parent → hub. Body ends with a live **Related notes** footer (dataviewjs: every note sharing a `topic/*` tag). |
| `SourceTemplate` | Capture in `Inbox/` | **Share-aware.** Guard skips notes already stamped `Type: Source` (Web Clipper); extracts the first URL from the body if present (Android share) instead of prompting; uses the filename as title unless "Untitled". Frontmatter written via API (`Type: Source`, `reading_status: inbox`, `source:`). Also auto-applied by Templater's folder template to any new file in `Inbox/`. Tags deferred to the Idea note. |
| `TasksTemplate` | Kanban board in a project folder | Inherits the project's `area/*` tag from `00000.md`; `Type: Tasks`, `kanban-plugin: board`, Backlog / In Progress / Done lists. |
| `DailyNoteTemplate` | Daily health log | `Type: Daily`, `tags: [area/health]`; Modal Forms `master_health_log` unchanged. |

---

## 4. Task convention

Kanban cards remain the **source of truth for status** (which list the card sits in). Convention, not requirement:

- If a card contains a wikilink, the **first link** is the note that *represents* that task; the Index renders it as the clickable task title.
- Cards without links fall back to the card text as title.
- Due dates use the Kanban plugin's `@{YYYY-MM-DD}` syntax, parsed by regex in the Index queries.

`Index.md` renders per-list tables (In Progress, Backlog) by flattening `file.tasks` across `PROJECTS/` and reading `meta(Tasks.section).subpath`; the title column is `choice(length(Tasks.outlinks) > 0, Tasks.outlinks[0], split(Tasks.text, " @")[0])`. `Boards.md` only lists the boards themselves.

---

## 5. Index.md layout

Top-to-bottom: **Tasks** (In Progress, Backlog) → **Projects by area** (dataviewjs grouped on the `area/*` namespace; untagged projects fall into `unsorted`) → **Areas** (hub notes) → **Reading queue** (WIP counter that flips to a warning above 10 undigested sources; inbox/reading with age; **stale list** for sources older than 14 days, oldest first; integrated) → **Ideas** (seeds/developing needing work, plus a serendipity block that surfaces 3 random Idea notes on every open) → **Notes by topic** (dataviewjs grouped on `topic/*`; replaces the old Category→Subcategory report; multi-topic notes appear once per topic by design) → **Resources** → **Archived projects** → **Audit** (notes without `Parent`, project indexes missing an `area/*` tag).

---

## 6. Plugin stack

Unchanged from v1 — see `.obsidian/community-plugins.json`. Load-bearing: **Templater** (template JS), **Dataview/DataviewJS** (all dashboards), **Kanban** (boards), **Modal Forms** (health log), **Homepage** (opens `Index.md` on startup). Supporting: periodic-notes, quickadd, obsidian-git, system3-relay, linter, outliner, various-complements, find-unlinked-files, spaced-repetition, paste-image-rename, better-export-pdf, pdf-plus.

---

## 7. Workflows

Capture setup and per-device usage (QuickAdd, phone share sheet, Web Clipper): see [[Capture — Setup and Usage]].

- **New area:** create note in `AREAS/` from `AreaIndexTemplate`. That's the whole operation — no folder.
- **New project:** create folder in `PROJECTS/`, add `00000.md` from `ProjectIndexTemplate`, optionally a board from `TasksTemplate`.
- **Capture (in-vault):** press `Cmd/Ctrl+Shift+S` — the QuickAdd "New Source" command creates a stamped Source note in `Inbox/` and prompts for title + URL. The Web Clipper covers capture from the browser.
- **Capture (phone, share sheet):** new notes default to `Inbox/`, and a Templater folder template auto-runs `SourceTemplate` on any new file there. Share a page → Obsidian creates the note → the template detects the URL in the body and the title in the filename and stamps the full Source frontmatter. A guard skips notes that arrive already stamped (Web Clipper). Caveat: Templater may skip auto-applying to files created with content — if a shared note comes through raw, insert `SourceTemplate` manually; the same detection logic runs.
- **Capture → integrate (intake v1, unchanged):** clip/capture → `Inbox/` `Type: Source` → read & highlight → write `Type: Idea` note(s) in your own words (tags assigned here), backlink both ways, flip `reading_status: integrated`.
- **New task with a note:** add a card to the project board; if the task deserves a note, create the note (any type) and put its wikilink first in the card.
- **Develop ideas:** new Idea notes start as `status: seed` and sit in the Index's "Seeds needing development" table until promoted to `developing`, then `evergreen` (edit the field by hand). The related-notes footer on each Idea note and the serendipity block on the Index exist to trigger collisions between ideas.
- **Promote a topic to a hub (rule of thumb, not machinery):** when a `topic/*` tag accumulates ~10+ notes, create a hub note for it — same pattern as an area hub, with queries filtered on that topic tag. Maps of Content emerge from evidence; don't create them upfront.
- **Inbox discipline:** the reading queue shows a hard number against a cap of 10. If the warning is showing, digest or delete before capturing more. Anything older than 14 days appears in the stale list — integrate it or admit you never will and delete it.
- **Archive a project:** move the folder to `ARCHIVE/`. Tags travel with the files; the Index picks it up automatically.

## 8. Migration notes (v1 → v2, 2026-07-12)

The PARA folders were empty at migration time, so no note frontmatter needed conversion. Changed in one pass: all 8 templates, `Index.md`, `Boards.md`, `Inbox/00000.md`, this file. The v1 `Area`/`Category`/`Subcategory` contract, the AREAS folder-per-area layout, and the old grouped Category report are retired. The intake-plan documents (`Information Intake — v1 *`) still describe the old `Area` fields in places; their Source/Idea pipeline is untouched and remains valid.
