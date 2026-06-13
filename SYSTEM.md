---
Parent: "[[Index|Link]]"
Type: Documentation
Area: Meta
---
# SecondBrainPARA — Technical System Reference

Strictly technical description of the vault's architecture, metadata contract, template logic, query layer, plugin stack, configuration, and supported workflows. The vault implements the **PARA** method (Projects, Areas, Resources, Archive) over plain Markdown, with all dynamic behaviour pushed into frontmatter conventions and plugin queries. The four top-level PARA folders are *structural scaffolding only* — they contain no logic. All intelligence lives in `Index.md`, `Boards.md`, the seven files under `Templates/`, and the plugin configuration under `.obsidian/`.

---

## 1. Plugin stack

### 1.1 Load-bearing plugins

The system is non-functional without these five. Each is structurally required:

| Plugin | Role in the system | Where invoked |
|---|---|---|
| **Templater** (`templater-obsidian`) | Executes JS (`<%* … %>`) at file creation: renames files, prompts the user, reads sibling frontmatter, and writes frontmatter via `app.fileManager.processFrontMatter`. | All 7 templates |
| **Dataview** | Declarative `dataview` query blocks (TABLE/LIST) that index frontmatter across the vault. | `Index.md`, `Boards.md`, Area/Project index templates |
| **DataviewJS** | Imperative `dataviewjs` block for the grouped Category→Subcategory report (needs procedural grouping Dataview's DQL can't express). | `Index.md` |
| **Modal Forms** (`modalforms`) | `app.plugins.plugins.modalforms.api` — renders the `master_health_log` form and serialises answers to frontmatter. Global namespace `MF`, editor opens right. | `DailyNoteTemplate.md` |
| **Kanban** (`obsidian-kanban`) | Reads `kanban-plugin: board` frontmatter + the `%% kanban:settings %%` block to render a board view from Markdown lists. | `TasksTemplate.md`, surfaced by `Boards.md` |

### 1.2 Full installed inventory

All 17 community plugins enabled in `.obsidian/community-plugins.json`, with their role in this vault:

| Plugin | Function |
|---|---|
| `dataview` | Query engine (above). |
| `templater-obsidian` | Templating engine (above). |
| `obsidian-kanban` | Kanban boards (above). |
| `modalforms` | Structured input forms (above). |
| `homepage` | Opens `Index.md` on startup as the vault home (configured "Main Homepage", `openOnStartup: true`, replace-all-panes). Makes the Index the default landing dashboard. |
| `periodic-notes` | Daily/periodic note scheduling; pairs with `DailyNoteTemplate.md` to create dated health-log notes. |
| `quickadd` | Capture/macro launcher for fast note creation. AI assistant present but `disableOnlineFeatures: true`. |
| `obsidian-git` | Commits the vault to Git for backup/version history (`commitMessage: "vault backup: {{date}}"`, `syncMethod: merge`). One of the sources of `.sync-conflict` artefacts. |
| `system3-relay` | Real-time multi-device collaboration/sync layer — the primary generator of the `.sync-conflict-*` files seen across `.obsidian/` and surfaced by the Index audit. |
| `obsidian-outliner` | Structured list/outline editing (bullet manipulation, fold). |
| `obsidian-linter` | Markdown/YAML normalisation on save; in this vault most YAML-mutating rules are disabled to avoid clobbering template-written frontmatter. |
| `various-complements` | Word/phrase autocompletion while typing. |
| `find-unlinked-files` | Detects orphan / unlinked / broken-link files — a GUI complement to the Index's "Files without parent links" query. |
| `obsidian-spaced-repetition` | Flashcard review / SRS over note content. |
| `obsidian-paste-image-rename` | Auto-renames pasted images on paste for consistent attachments. |
| `better-export-pdf` | Bulk/configurable PDF export of notes. |
| `pdf-plus` | Enhanced PDF viewing & annotation/back-linking. |

Core plugins enabled include `daily-notes`, `templates` (folder set to `Templates`), `graph`, `backlink`, `outgoing-link`, `bookmarks`, `file-recovery`, and `bases`. The legacy core `sync` is **off** — syncing is handled by `system3-relay` and `obsidian-git` instead.

---

## 2. Folder topology

```
SecondBrainPARA/
├── Index.md            ← global dashboard (Dataview hub, opens on startup via Homepage)
├── Boards.md           ← cross-project task dashboard (flattens all Kanban boards)
├── AREAS/              ← long-running responsibilities (one subfolder per Area)
├── PROJECTS/           ← active, outcome-bound efforts (one subfolder per Project)
├── RESOURCES/          ← topic reference material
├── ARCHIVE/            ← completed / inactive projects
├── Templates/          ← Templater source templates (7)
└── .obsidian/          ← vault configuration: plugins, settings, themes, sync state
```

PARA semantics as implemented here:

- **Area** — a sphere of ongoing responsibility (e.g. `Health`). An Area folder is identified by its `00000.md` index file with `Type: Area`. Projects and Resources link *up* to an Area.
- **Project** — a folder under `PROJECTS/` whose `00000.md` carries `Type: Project`. Holds many *ProjectFiles*. Linked up to one Area.
- **Resource** — reference note under `RESOURCES/`, `Type: Resource`, classified by `Area` + `Category` + `Subcategory`.
- **Archive** — structurally identical to Projects; the move to `ARCHIVE/` is the only thing that marks a project inactive. Queries discriminate purely on folder (`FROM "ARCHIVE"`).

### The `00000.md` convention

Every Area, Project, and Archived Project folder contains exactly one file named **`00000`**. This is the folder's **index/anchor note**. Three mechanisms depend on it:

1. **Discovery** — Dataview queries use `WHERE file.name = "00000"` to list "one row per Area/Project" instead of one row per note.
2. **Parent resolution** — child templates build their `Parent` wikilink by pointing at `<folder>/00000` (e.g. `[[PROJECTS/MyProject/00000|Link]]`).
3. **Metadata inheritance** — `ProjectFile`, `Resource`, and `Tasks` templates *read* `Area`/`Category` out of the sibling `00000.md` frontmatter via `app.metadataCache.getFileCache(file)?.frontmatter`, so child notes inherit their parent's classification without re-prompting.

Templates that produce an index note force the title to `00000` (Area, Project, Archive). Templates that produce leaf notes prompt for a real title (Idea, Resource, ProjectFile) or derive one (`<folder> Tasks`).

---

## 3. Frontmatter contract (metadata schema)

The query layer is only as good as the frontmatter every note carries. Canonical keys:

| Key | Type | Set by | Meaning |
|---|---|---|---|
| `Parent` | wikilink `[[…/00000\|Link]]` | all templates | Upward link to the containing index note. Powers the "Files without parent links" integrity check. |
| `Type` | enum | all templates | One of `Area`, `Project`, `ProjectFile`, `Resource`, `Idea`, `Tasks`. The system's primary discriminator. |
| `Area` | string | all templates | Owning Area. On Areas it is the folder name; elsewhere inherited from the parent `00000`. |
| `Project` | string | Project/ProjectFile/Tasks | Owning project (= folder name). |
| `Category` | string | Project/ProjectFile/Resource | Top-level classification (free-text, chosen from existing values or "Other…"). |
| `Subcategory` | string | ProjectFile/Resource | Second-level classification, drives the Index Category report. |
| `tags` | list | Project/Idea/Resource | Initialised empty `[]` for later tagging. |
| `kanban-plugin` | `"board"` | Tasks | Signals the Kanban plugin to render the file as a board. |

`Category` and `Subcategory` are **vault-global free vocabularies**: templates harvest the *distinct existing values* across all files (`app.vault.getFiles()` → collect `frontmatter.Category`/`.Subcategory` into a `Set`) and present them in a `tp.system.suggester` dropdown with an "Other…" escape hatch that prompts for a new value. This produces a self-reinforcing, drift-resistant taxonomy without a central schema file.

---

## 4. Templates — behavioural reference

All templates are Templater scripts. Common idioms:
- **Title handling**: if `tp.file.title` starts with `"Untitled"`, either set it to `"00000"` (index notes) or `await tp.system.prompt("Title")` (leaf notes), then `await tp.file.rename(title)`.
- **Deferred frontmatter write**: leaf/index writes are wrapped in `setTimeout(… , 200)` to let Obsidian finish creating the file before `processFrontMatter` mutates it (a race-condition workaround).
- **Folder = identity**: `tp.file.folder()` is used as the Project/Area name throughout, so *moving/creating a note in the right folder* is what assigns its classification.

### 4.1 AreaIndexTemplate
Produces an Area's `00000`. Forces title to `00000`, sets `Parent=[[Index|Link]]`, `Type=Area`, `Area=<folder name>`. Body renders three Dataview blocks scoped to this Area: child **Projects** (`FROM "PROJECTS" WHERE file.name="00000" AND Area=<folder>`), **Ideas** (notes in this folder with `Type=Idea`), and **Resources** (`FROM "RESOURCES" WHERE Area=<folder>`). Net effect: each Area note is a live mini-dashboard of everything attached to that responsibility.

### 4.2 ProjectIndexTemplate
Produces a Project's `00000`. Prompts the user to pick the owning **Area** (suggester over distinct `Area` values found under `AREAS/`) and a **Category** (global vocabulary + "Other…"). Writes `Parent=[[AREAS/<area>/00000|Link]]`, `Type=Project`, `Area`, `Category`, `Project=<folder>`, `tags=[]`. Body lists all non-`00000` notes in the project folder (`LIST WHERE contains(file.folder, this.file.folder) AND file.name != "00000"`).

### 4.3 ProjectFileTemplate
A leaf note inside a project. Prompts for a title, then **inherits** `Area` and `Category` by reading the sibling `<folder>/00000.md` frontmatter (no re-prompt), and prompts only for `Subcategory` (global vocabulary + "Other…"). Writes `Parent=[[PROJECTS/<folder>/00000|Link]]`, `Type=ProjectFile`, inherited `Area`/`Category`, chosen `Subcategory`, `Project=<folder>`.

### 4.4 TasksTemplate
Creates the project's Kanban board. Title auto-derived as `"<folder> Tasks"`. Inherits `Area` from the project `00000`. Sets `Parent`, `Type=Tasks`, `Project`, and crucially `kanban-plugin: board`. Body seeds three lists — **Backlog / In Progress / Done** — plus the `%% kanban:settings %%` block (`list-collapse`, `show-checkboxes:false`). The Kanban plugin then renders this file as a draggable board; the underlying storage remains plain Markdown lists.

### 4.5 IdeaTemplate
Lightweight capture note that lives inside an **Area** folder. Prompts for a title; sets `Parent=[[AREAS/<folder>/00000|Link]]`, `Type=Idea`, `Area=<folder>`, `tags=[]`. Surfaced by the Area index's Ideas query.

### 4.6 ResourceTemplate
Reference note under `RESOURCES/`. Prompts for title, then picks **Area** (suggester over `AREAS/`), **Category** (global), and **Subcategory** (global). Writes `Parent=[[AREAS/<area>/00000|Link]]`, `Type=Resource`, `Area`, `Category`, `Subcategory`, `tags=[]`.

### 4.7 DailyNoteTemplate
The only **static-frontmatter** template (keys written literally, not via JS): `Type=ProjectFile`, `Area=Health`, `Category=Daily note`, `Subcategory=health`, `Project=Daily Notes`, parented to `PROJECTS/Daily notes/00000`. On creation it opens the **`master_health_log` Modal Form** via the Modal Forms API and appends the result as a frontmatter string (`result.asFrontmatterString()`) under a `## Feelings` heading, followed by a free-text `## Notes` section. This makes daily notes a structured health-logging instrument whose form fields become queryable frontmatter.

---

## 5. Query layer — `Index.md` dashboard

`Index.md` is the vault's control panel. It composes the metadata contract into nine live views:

1. **Boards** — a wikilink hub (`[[Boards]]`) that resolves to `Boards.md` (see §5.1), the cross-project task dashboard.
2. **Areas** — `TABLE … FROM "AREAS" WHERE file.name = "00000"` → one row per Area.
3. **Projects** — `FROM "PROJECTS" WHERE file.name = "00000"`, showing Project + Area → active projects.
4. **Resources** — `TABLE Area FROM "RESOURCES"` → all reference notes by Area.
5. **Archived projects** — same shape as Projects but `FROM "ARCHIVE" WHERE file.name="00000"`.
6. **Categories and subcategories** — a **DataviewJS** block: runs a DQL query grouped by `Category`, `FLATTEN rows as R`, then in JS builds a dictionary keyed by Category and emits one sub-table (Note link + Subcategory) per Category. Procedural because DQL alone can't render "a separate table per group."
7. **Inactive projects** — `TABLE Project, Area FROM "ARCHIVE"` (all archive notes, not just indexes).
8. **Files without parent links** — integrity audit: `FROM "" WHERE !Parent AND !contains(file.name,"Template") AND file.name != "Index"`. Flags orphan notes that escaped the template pipeline.
9. **Sync conflicted files** — operational hygiene: `WHERE contains(file.name, ".sync-conflict")` surfaces Obsidian Sync / file-sync collision artefacts for cleanup.

Queries 8 and 9 are the system's **self-maintenance layer**: they make schema violations and sync damage visible on the home screen rather than letting them rot silently. (Note: the sync-conflict audit only catches Markdown conflicts; `.sync-conflict` artefacts inside `.obsidian/` are JSON and are not indexed by Dataview — clean those manually.)

### 5.1 `Boards.md` — cross-project task dashboard

`Boards.md` is the target of the Index's `[[Boards]]` link and is the operational view *across all Kanban boards at once*. It contains three Dataview blocks:

1. **In progress tasks** — `FROM "" FLATTEN file.tasks AS Tasks WHERE contains(meta(Tasks.section).subpath, "In Progress")`. Aggregates every checklist item that sits under an `In Progress` heading in *any* file, regardless of which board it lives on.
2. **Kanban boards** — `FROM "PROJECTS" WHERE contains(file.name, " Tasks")`. Lists every task board (board files are named `<Project> Tasks` by TasksTemplate), giving a project-level index of boards.
3. **Backlog tasks** — identical to #1 but filtered on the `Backlog` section.

The task rows are parsed, not stored structured. Each block:
- `split(Tasks.text, "@{")[0]` → the task label (text before the due-date token).
- `regexreplace(Tasks.text, "[^0-9-]", "")` → the **due date**, extracted by stripping everything but digits and hyphens.
- `meta(Tasks.section).subpath` → the **status** (the Kanban column heading the task lives under).
- `file.link` → the originating **board**.

This establishes a **task-syntax convention**: inside a board, a task is written as a Markdown checklist item carrying a `@{YYYY-MM-DD}` due-date token (e.g. `- [ ] Draft proposal @{2026-06-20}`). The Kanban plugin treats the columns (`Backlog`/`In Progress`/`Done`) as section headings, and `Boards.md` mines those sections vault-wide. Net effect: Kanban gives the per-project board UI; `Boards.md` gives the portfolio-wide "what's in progress / what's queued, and when is it due" rollup — without any extra metadata beyond the inline `@{date}` token.

---

## 6. Data model (relationships)

```
Index ──[[Boards]]──> Boards.md ──(Dataview flatten of file.tasks across vault)
  │
  └──< Area(00000) ──< Project(00000) ──< ProjectFile
                │                  └─────────< Tasks (Kanban board) ──> tasks w/ @{date}
                ├──< Idea
                └──< Resource (also FROM RESOURCES/, keyed by Area)

ARCHIVE/Project(00000) ── structurally identical to PROJECTS; inactive by location
```

- Containment is **physical** (folder) and **logical** (`Parent` wikilink + `Area`/`Project` strings) simultaneously; queries exploit whichever is cheaper.
- Classification is **two-axis**: PARA *location* (Project vs Area vs Resource vs Archive) is orthogonal to topical *taxonomy* (`Category` → `Subcategory`). A note has both at once.
- Inheritance flows **downward** from `00000` index notes to leaf notes at creation time and is then *frozen* into the leaf's frontmatter (denormalised), so later moving the parent does not retroactively update children.

---

## 7. Workflows

**Stand up a new Area.** Create a subfolder under `AREAS/` → create a note (Templater fires AreaIndexTemplate) → it becomes `00000` with `Type=Area`. The Area note immediately shows empty Projects/Ideas/Resources tables that fill as you attach items.

**Start a project.** Create a folder under `PROJECTS/` → new note → ProjectIndexTemplate prompts for Area + Category → `00000` created and linked up to the Area. The Area's dashboard and the Index "Projects" table now list it automatically.

**Add work to a project.** Inside the project folder, create notes → ProjectFileTemplate inherits Area/Category from `00000`, prompts only for Subcategory. The project `00000` lists them via its child-files query.

**Track project tasks.** Create the Tasks note (TasksTemplate) → a Kanban board with Backlog/In Progress/Done. Add cards as checklist items with an optional `@{YYYY-MM-DD}` due-date token; drag between columns. Data persists as Markdown lists. Open `Boards.md` (via `[[Boards]]` on the Index) to see, across *every* project, all In Progress and Backlog tasks with parsed due dates and their source board — a portfolio-wide task view that needs no extra metadata.

**Capture an idea.** In an Area folder, new note → IdeaTemplate → tagged `Type=Idea`, surfaced under that Area's Ideas list.

**File a resource.** Under `RESOURCES/`, new note → ResourceTemplate → choose Area + Category + Subcategory. Appears in the Area's Resources list, the Index Resources table, and the Category/Subcategory report.

**Daily health logging.** Create today's daily note → DailyNoteTemplate opens the `master_health_log` Modal Form → answers stored as frontmatter under `## Feelings`, free notes under `## Notes`. Because the answers are frontmatter, they are queryable/aggregatable across days.

**Archive a project.** Move the project folder from `PROJECTS/` to `ARCHIVE/`. No metadata edit needed — the Index's "Archived/Inactive projects" queries pick it up by folder and it drops out of the active Projects table.

**Routine maintenance.** Check the Index's **Files without parent links** (fix orphans by re-applying the right template / adding `Parent`) and **Sync conflicted files** (resolve `.sync-conflict` duplicates) sections to keep the metadata graph and storage clean. The `find-unlinked-files` plugin offers the same orphan detection as a GUI command.

---

## 8. Vault configuration (`.obsidian/`)

The configuration directory pins the runtime behaviour the templates and queries assume:

- **Startup / homepage** — `homepage` plugin is set to open `Index.md` on launch ("Main Homepage", replace all panes), so the Dataview dashboard is the landing screen.
- **Templater** (`plugins/templater-obsidian/data.json`) — `templates_folder: "Templates"`, `trigger_on_file_creation: true` (templates fire automatically on new-file creation, enabling the `00000` auto-index pattern), `enable_system_commands: true`, `enable_folder_templates: true`, `command_timeout: 5`, `auto_jump_to_cursor: true`. The folder-template list is currently a single empty pair, i.e. folder→template mapping is not pre-wired, so template selection is effectively manual/per-creation.
- **Core Templates** (`templates.json`) — folder also set to `Templates`.
- **Modal Forms** — global namespace `MF`, editor position right. The `master_health_log` form referenced by DailyNoteTemplate must exist in this plugin's `formDefinitions` for daily notes to work (the shipped config also contains an `example-form`).
- **Linter** (`obsidian-linter`) — YAML-mutating rules (`format-yaml-array`, `insert-yaml-attributes`, `sort-yaml-array-values`, etc.) are **disabled**, deliberately preventing the linter from rewriting the frontmatter that templates author.
- **PDF export** (`app.json`) — Letter, portrait, zero margin, 100% scale, includes note name; consumed by `better-export-pdf` / `pdf-plus`.
- **Linking** — `alwaysUpdateLinks: true`, so renaming/moving notes rewrites wikilinks automatically (important given the `Parent`/`00000` link graph).
- **Sync & versioning** — handled by `system3-relay` (real-time multi-device) and `obsidian-git` (commit-based backup); core `sync` is off. The numerous `*.sync-conflict-*.json` files under `.obsidian/` are collision artefacts from concurrent multi-device edits to settings — harmless to delete, and a signal of how actively the vault is synced across desktop and mobile (`workspace-mobile.json` confirms mobile use).

---

## 9. Design properties & constraints

- **Convention over configuration**: behaviour is encoded in folder placement + frontmatter, not in a settings file. Put a note in the right folder and the template does the rest.
- **Denormalised inheritance**: child classification is copied at creation, not referenced live — fast queries, but parent changes don't propagate.
- **Free-vocabulary taxonomy** with suggester + "Other…" keeps `Category`/`Subcategory` consistent without a controlled-vocab file, at the cost of possible near-duplicates (e.g. casing) that only manual review catches.
- **Single-anchor coupling**: everything hinges on the `00000` convention; a missing or misnamed anchor breaks Parent links and per-folder rollups.
- **`setTimeout` race workaround** in templates is timing-dependent; on very slow vaults frontmatter writes could in principle miss.
- **Portable core, plugin-bound dynamics**: notes are plain Markdown (portable), but every dashboard, board, form, and inheritance step requires its plugin to be present and enabled.
