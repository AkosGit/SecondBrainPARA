---
Parent: "[[Index|Link]]"
Type: Documentation
Area: Meta
---
# Information Intake — v1 Technical Implementation Plan

A concrete, build-it-this-afternoon plan for **v1** of the information-intake system described in [[Information intake]], implemented on the existing [[SYSTEM|SecondBrainPARA architecture]]. This document is the engineering spec: exact schema, exact files to create, exact plugin settings, exact Dataview queries, and an acceptance test. It deliberately implements *only* v1 — the deferred features and their triggers are listed at the end so you know what you are **not** building yet.

> **Scope contract.** v1 = one folder, one new `Type` value, one `reading-status` field (three states), one Templater template, one capture-tool config, one settings toggle, and three dashboard queries. The unit of "done" for a source is: **a `Type: Idea` note exists, in your own words, that links back to it.** Everything else is v2.

---

## 1. The one rule this hangs on

A source is **not** finished when you have read and highlighted it. It is finished only when it has produced a new `Type: Idea` note, in your own words, that links back to the source. Highlighting is a cheap filter that points at where the real work is owed; it is not the work. The whole status model and every dashboard below exist to make that one transition — *source → idea* — the cheapest and most visible thing in the vault.

Mechanically: a source reaches `reading-status: integrated` when at least one Idea note backlinks to it. In v1 you flip that field by hand at the moment you write the Idea note. (Enforcing it automatically from an `ideas-extracted` list is v2.)

---

## 2. How v1 sits on the existing vault

v1 adds a thin layer on top of the contract already documented in [[SYSTEM]]. Nothing is replaced.

| Existing mechanism (SYSTEM.md) | What v1 does with it |
|---|---|
| `Type` enum (`Area`, `Project`, `ProjectFile`, `Resource`, `Idea`, `Tasks`) | **Adds one value: `Source`.** Reuses existing `Idea` unchanged. |
| `reading-status` — **does not exist yet** | **New field**, used *only* for the reading lifecycle. Never reuse the vault's `status` (draft/review/evergreen) — a bare `status` would collide and corrupt both query families. |
| `Parent` wikilink → folder `00000` anchor | Source notes parent to a new `Inbox/00000` anchor (see §4, Step 1). Keeps the Index "Files without parent links" audit clean. |
| Templater (`trigger_on_file_creation: true`) | One new template, built in the same idiom as the existing seven. |
| Dataview / DataviewJS on `Index.md` | Three new query blocks added to `Index.md`. |
| `Type: Idea` + `IdeaTemplate` | Reused as-is for the extracted permanent note; v1 adds one manual backlink line (`source-notes`). |
| `obsidian-spaced-repetition` (installed, enabled) | One settings toggle so your `==highlights==` don't silently become flashcards. |
| Web Clipper, pdf-plus, QuickAdd, Modal Forms | Web Clipper + pdf-plus + QuickAdd are configured in v1. Modal Forms is v2. |
| Index "Files without parent links" audit | Reconciled — see §4 Step 1. Sources will **not** pollute it. |

**Two deliberate divergences from [[Information intake]] §8, documented so they don't look like mistakes:**

1. **Parent is stamped at capture, not left blank.** §8.A defers `Parent` to the digest step (blank at capture). Left blank, every un-digested Source would surface in the Index's *Files without parent links* integrity audit (`FROM "" WHERE !Parent …`), conflating "needs digesting" with "schema orphan." v1 instead stamps `Parent: "[[Inbox/00000|Link]]"` at capture against a tiny Inbox anchor note. The audit stays meaningful, at the cost of one extra file.
2. **The Source's own `Area`/`Category` stay blank in v1.** PARA classification lives on the **Idea** note (which is properly parented to its Area via the existing `IdeaTemplate`). The Source is just a lightweight reading index in `Inbox/`. This keeps capture a single action and defers all classification machinery — exactly the report's intent — without a second pipeline.

---

## 3. Schema

### 3.1 `Type: Source` frontmatter (v1)

| Key | Type | Set by | v1 value / behaviour |
|---|---|---|---|
| `Type` | enum | template (fixed) | `Source` |
| `Parent` | wikilink | template (fixed) | `"[[Inbox/00000\|Link]]"` |
| `reading-status` | enum | template = `inbox`; you advance | `inbox` → `reading` → `integrated` |
| `source` | url / path | Web Clipper / QuickAdd | original URL, or vault path to the PDF |
| `author` | text / list | Web Clipper (blank on manual) | one or many authors |
| `ideas-extracted` | list | seeded `[]`; optional in v1 | wikilinks to Idea notes (the backlink is the real record) |
| `tags` | list | seeded `[]` | vault tag contract |

Capture date needs **no key** — `file.ctime` is implicit on every note and is what the queue and age columns read. `Area` / `Category` / `Subcategory` / `published` / `source-type` are intentionally omitted from v1 (they appear in §8 of the report as later additions).

### 3.2 `reading-status` — three states

```
   capture          you open it        you write an Idea note
 ──────────► inbox ──────────► reading ──────────► integrated
```

| State | Meaning | How it's set |
|---|---|---|
| `inbox` | Captured, not yet opened. The collector's-fallacy danger zone. | Automatically, at capture. |
| `reading` | Actively reading / highlighting. | You set it by hand when you start. |
| `integrated` | At least one `Type: Idea` note links back. **This is "done."** | You set it by hand when you write the Idea note. |

There is no separate `done` token — `integrated` *is* done, so v1's three states are a strict subset of the report's full vocabulary and every query below keeps working if you later add `processed`/`dropped` in v2. There is no WIP cap and no staleness alarm in v1 — just a count (§6).

### 3.3 Idea note — one addition

The extracted note uses the **existing** `Type: Idea` + `IdeaTemplate` unchanged (it already sets `Parent` → Area `00000`, `Area`, `tags: []`). v1 adds a single backlink so the source↔idea relationship is queryable from both ends:

- On the **Idea** note, add `source-notes:: [[<the Source note>]]` (an inline field is fine), or simply wikilink the Source in the body.
- On the **Source** note, add that Idea's wikilink to `ideas-extracted` and flip `reading-status: integrated`.

Cloning `IdeaTemplate` into a reading-aware variant is a v2 nicety; in v1 a single typed line does the job.

---

## 4. Build steps (≈ one afternoon)

Do these in order. Steps 1–6 are one-time setup; Step 7 is the habit.

### Step 1 — Create `Inbox/` and its anchor

1. Create a top-level folder `Inbox/`. Keep it **flat** — no sub-folders, no tags on the folder. It is a triage tray, not an archive.
2. Inside it create `Inbox/00000.md` with this content, so captured Sources have a valid `Parent` target and stay out of the orphan audit:

```yaml
---
Parent: "[[Index|Link]]"
Type: Documentation
Area: Meta
---
# Inbox

Staging lane for captured `Type: Source` notes awaiting digest. Flat by design.
See [[Information Intake — v1 Implementation Plan]].
```

> Alternative if you'd rather not add the anchor: leave `Parent` blank on Sources and patch the Index audit query to `… AND !startswith(file.folder, "Inbox")`. The anchor is recommended because it needs no query edits and matches the vault's universal `00000` convention.

### Step 2 — Create the Source template

Create `Templates/SourceTemplate.md`. This is the eighth template, built in the same idiom as the existing seven (prompt → rename → write frontmatter). Note `ideas-extracted` and `Area`/`Category` are intentionally minimal for v1.

````markdown
<%*
// New Source note — lands in Inbox/, classification deferred to the Idea note.
const title = await tp.system.prompt("Source title");
const url    = await tp.system.prompt("URL or PDF path", "");
if (tp.file.title.startsWith("Untitled")) {
  await tp.file.rename(title);
}
-%>
---
Type: Source
Parent: "[[Inbox/00000|Link]]"
reading-status: inbox
source: <% url %>
author:
ideas-extracted: []
tags: []
---

# <% title %>

## Highlights
<%* /* ==highlights== (web) or pdf-plus backlinks (papers) land here while reading */ -%>

## Digest — owes an Idea note
- [ ] Cluster highlights by concept (not by the source's chapter order)
- [ ] Write Idea note(s) in your own words; backlink this Source
- [ ] Add the Idea link to `ideas-extracted`, set `reading-status: integrated`
````

The embedded checklist is itself a tiny forcing function: the Source note tells you what it still owes. Confirm Templater's `templates_folder` is `Templates` (it is, per [[SYSTEM]] §8) so this file is picked up.

### Step 3 — Wire a one-action "New Source" command (QuickAdd)

For manual captures (PDFs, or "I want a Source note without the whole article body"):

1. QuickAdd → Manage Macros / Add Choice → **Template** choice named **"New Source."**
2. Template path → `Templates/SourceTemplate.md`.
3. File name format → `{{VALUE}}` (or leave Templater's prompt to rename).
4. Target folder → `Inbox/`.
5. Add the choice to the command palette / a ribbon icon. One command now creates a stamped Source note in `Inbox/`.

### Step 4 — Configure the Obsidian Web Clipper (the bulk web on-ramp)

In the Web Clipper extension, create/edit a template so a clipped article lands already speaking the schema:

- **Output folder:** `Inbox/`
- **Note content:** full article (Markdown). In-browser highlights arrive as `> blockquotes`.
- **Properties (frontmatter)** — set exactly:

```yaml
Type: Source
Parent: "[[Inbox/00000]]"
reading-status: inbox
source: {{url}}
author: {{author}}
tags: []
```

- **Delete** the clipper's default `category: Clippings` property (classification is deferred).
- Leave `Area`/`Category`/`Subcategory` unset.

> **Mobile boundary condition.** The clipper is desktop-first. On Android, Chrome/Brave block extensions (use Firefox/Kiwi or a "share → Firefox → clip" hop); on iOS it's Safari-only with occasionally flaky metadata. If you capture mostly on desktop, this is strictly fine. A mobile Readwise front-end is a v2 seam, not a v1 concern.

### Step 5 — PDF capture flow (pdf-plus + companion note)

PDFs are **not** converted to Markdown. The PDF is the record; a sidecar Source note is its reading index.

1. Drop the PDF into the vault (e.g. `Inbox/`).
2. Run **"New Source"** (Step 3) to create its companion Source note; set `source:` to the PDF's vault path.
3. Highlight in the PDF with pdf-plus. Each highlight becomes a Markdown backlink of the form `[[paper.pdf#page=3&selection=…&color=yellow]]` — paste these into the Source note's `## Highlights`. The annotations are plain Markdown and survive even if the plugin dies.

> **Sync caution:** the PDF binary lives in the vault and adds payload to `system3-relay` + `obsidian-git` sync. Annotate a given paper from **one device at a time** to avoid `.sync-conflict` artefacts (which your `Index.md` already surfaces).

### Step 6 — Stop `==`/`**` from becoming flashcards

`obsidian-spaced-repetition` is already enabled and, at defaults, treats `==` and `**` as cloze-deletion markers — so your hand-typed highlights would silently turn into review cards. Fix it once now:

- Spaced Repetition settings → **disable `==`/`**` as cloze markers**; rely on `{{curly braces}}` for clozes instead (a v2 concern, but set the toggle now).

After this, `==text==` always means "process me" and never "test me." (Web Clipper blockquotes and pdf-plus backlinks never collided with cloze syntax; only your typed `==`/`**` did.)

### Step 7 — Add the dashboards to `Index.md`

Add the three blocks in §6 to `Index.md`, next to the existing *Files without parent links* query, so the reading backlog sits in the same field of view as the rest of the vault's health.

---

## 5. The digest habit (where learning actually happens)

This is the only recurring behaviour v1 asks of you. When you pick a Source off the queue:

1. **Read and filter.** Mark what matters with `==highlights==` (web) or pdf-plus colour highlights (PDF). Set the Source's `reading-status: reading` when you start. Highlight freely — it's cheap — but remember a highlight is a pointer, not a finish.
2. **Annotate as you go.** Next to a highlight, write the constructive thought it triggers ("but this contradicts X…", "this is why Y works"). That marginal note *is* the high-value move.
3. **Cluster.** Group your highlights/annotations into piles by **concept**, explicitly *not* by the source's chapter order.
4. **Write the Idea note(s).** For each keeper cluster, create a `Type: Idea` note (existing `IdeaTemplate`), in **your own words**:
   - **Concept-oriented title** — name the idea, not the source ("Retrieval beats restudy for delayed retention," not "Notes on Roediger 2006").
   - **Atomic as an outcome, not a gate** — one idea per note, but don't let perfectionism block capture.
   - **Link it** — at minimum back to the Source (`source-notes:: [[Source]]`), and ideally to one related Idea in a **different Area** (cross-domain links are the high-value ones).
5. **Close the loop.** Add the Idea's wikilink to the Source's `ideas-extracted`, then set the Source's `reading-status: integrated`. The Source drops off the active queue.

Not every source earns an Idea note. The escape hatch is honest abandonment — in v1 that simply means deleting the capture (the `dropped` *state* is a v2 refinement; deleting guilt-free is the v1 equivalent). The rule is "every highlight you **keep** owes an Idea note," never "every article you open must produce one." Your processing rate should look like a power law — that's correct, not a failure.

Do **not** let an AI summarizer (the clipper's Interpreter, etc.) perform this generative step for you — that step is the entire point. Use AI, if at all, as a pre-read filter only.

---

## 6. Dashboards (copy-paste into `Index.md`)

> **Obsidian note:** the blocks below are shown inside four-backtick fences so they render as *text* in this plan note rather than executing. When you add them to `Index.md`, copy only the inner three-backtick block (```` ```dataview … ``` ````).

**Why the `WHERE reading-status AND …` guard matters:** Dataview *silently drops* a note whose `WHERE` field is missing rather than erroring. The leading guard is what keeps an un-stamped capture from vanishing from the counts. Keep it on every query.

### 6.1 The reading queue (oldest-first, so it drains FIFO)

````
```dataview
TABLE WITHOUT ID
  file.link AS "Source",
  reading-status AS "Status",
  source AS "URL",
  (date(today) - file.ctime).days + " d" AS "Age"
FROM "Inbox"
WHERE Type = "Source" AND reading-status AND reading-status = "inbox"
SORT file.ctime ASC
```
````

### 6.2 The inbox count (the v1 forcing function — a number, not an alarm)

````
```dataview
TABLE WITHOUT ID length(rows.file.link) AS "Items waiting to be read"
FROM "Inbox"
WHERE Type = "Source" AND reading-status AND reading-status = "inbox"
GROUP BY true
```
````

A single honest number does the work. If it embarrasses you, you process; if not, no automated banner would change your behaviour. (A WIP cap and staleness alarm are deferred to v2 precisely because their thresholds are uncalibrated guesses.)

### 6.3 Digestion progress (the funnel — the only metric that measures *learning*)

Because v1 has three states, this one-glance funnel is worth including from the start:

````
```dataview
TABLE WITHOUT ID
  reading-status AS "Stage",
  length(rows.file.link) AS "Count"
FROM "Inbox"
WHERE Type = "Source" AND reading-status
GROUP BY reading-status
SORT length(rows.file.link) DESC
```
````

The truest health check still costs no query at all: **glance at your Idea folder. If it's growing, the system works.**

---

## 7. Acceptance test (prove v1 works end-to-end)

Run this smoke test once after setup. v1 is correctly installed when every step passes.

1. **Capture (web):** clip any article with the Web Clipper. → A note appears in `Inbox/` with `Type: Source`, `reading-status: inbox`, `Parent: [[Inbox/00000]]`, `source:` populated.
2. **Capture (manual/PDF):** run "New Source." → A stamped Source note is created in `Inbox/`; you were prompted for title + URL/path.
3. **Queue shows it:** open `Index.md`. → The new Source appears in the §6.1 queue; the §6.2 count incremented by the right number.
4. **Advance:** set the Source's `reading-status: reading`. → It leaves the queue (6.1) and moves to the `reading` row of the funnel (6.3).
5. **Integrate:** write a `Type: Idea` note in your own words, link it back, add it to `ideas-extracted`, set `reading-status: integrated`. → The Source moves to the `integrated` row; the Idea note exists with a backlink to the Source.
6. **Audit clean:** confirm the Source does **not** appear in the Index's *Files without parent links* query (Parent is set), and no new `.sync-conflict` files were produced.
7. **Highlight integrity:** confirm a `==highlighted==` phrase in a Source did **not** turn into a spaced-repetition card (Step 6 toggle worked).

If all seven pass, v1 is done. Use it for a few weeks before reading the deferral table below.

---

## 8. What v1 deliberately excludes (and the trigger to add each)

Restraint is the governing principle: the elaborate system becoming the procrastination is a bigger risk than missing an article. Each feature below is real and defensible, and each stays **off** until a specific felt pain forces it on.

| Deferred to v2+ | Add only when… |
|---|---|
| `processed` and `dropped` states + auto-enforced `ideas-extracted` | "read vs not read" stops showing you *where* ideas stall |
| WIP-cap + staleness-alarm dashboards (≈10 items / 14 days) | a plain inbox **count** has demonstrably failed to move you for 3+ months |
| Spaced repetition (`{{curly}}` clozes on Idea notes, inference-level prompts) | specific ideas keep slipping and you want them top-of-mind |
| `maturity` field on Idea notes (`seedling`/`budding`/`evergreen`) | the Idea folder is big enough that "which ideas are thin?" is a real question |
| Maps of Content (hub notes) | you hit the "too many related Ideas to juggle" squeeze point |
| pdf-plus "Quote in callout" + Better Search Views annotation workflow | you read enough papers that companion-note highlighting feels clumsy |
| Modal Forms structured capture; `source-type` enum; cloned reading-aware Idea template | a query genuinely needs to branch on source type, or linear prompts feel too thin |
| Readwise as a mobile capture front-end (Jinja2 export → native frontmatter) | your reading is genuinely phone-heavy and the clipper's mobile friction bites |
| Full Zettelkasten (Folgezettel IDs, ZK folder, atomicity gate) | you become a high-volume publisher (by Doto's test, unnecessary for learning) |

**Skip outright, at every version:** a separate fleeting-note tier (it's just a `Source` that's still `reading`); sub-folders/tags on the inbox; clipping everything; and AI auto-summarization as your *digest* step.

---

## 9. Quick reference

| Thing | Value |
|---|---|
| New `Type` value | `Source` |
| New field | `reading-status` ∈ {`inbox`, `reading`, `integrated`} |
| New folder | `Inbox/` (flat) + `Inbox/00000.md` anchor |
| New template | `Templates/SourceTemplate.md` |
| Capture (web) | Obsidian Web Clipper → `Inbox/`, stamps `Type: Source` + `reading-status: inbox` |
| Capture (PDF/manual) | QuickAdd "New Source" + pdf-plus highlights |
| Highlight marker | `==process me==` (and `> blockquote` from clipper, `[[file.pdf#…]]` from pdf-plus) |
| "Done" = | a `Type: Idea` note links back → flip Source to `integrated` |
| Settings toggle | disable `==`/`**` as spaced-repetition cloze markers |
| Dashboards | queue (6.1) + count (6.2) + funnel (6.3) on `Index.md` |
| Best single metric | is the Idea folder growing? |

---
*Source material: [[Information intake]] (research blueprint, esp. §8.F/§7.A) and [[SYSTEM]] (vault architecture contract). This plan implements Phase 1 only.*
