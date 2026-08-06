---
Parent: "[[Index|Link]]"
Type: Documentation
Area: Meta
---
# Information Intake — v1 Setup Checklist

Companion to [[Information Intake — v1 Implementation Plan]]. Records what was built automatically, the manual steps you still need to do (browser-side things the vault can't reach), and the acceptance test. Built on the [[SYSTEM|SecondBrainPARA architecture]].

---

## A. Done automatically (in the vault)

| Plan step | What was created / changed | File |
|---|---|---|
| Step 1 | `Inbox/` folder (flat) + anchor note | `Inbox/00000.md` |
| Step 2 | Source template (8th template) | `Templates/SourceTemplate.md` |
| Step 3 | QuickAdd **"New Source"** command → writes to `Inbox/` | `.obsidian/plugins/quickadd/data.json` |
| Step 6 | Disabled `==`/`**` as cloze markers, enabled `{{ }}` | `.obsidian/plugins/obsidian-spaced-repetition/data.json` |
| Step 7 | Reading queue + Inbox count + Digestion funnel | `Index.md` |

> **Reopen Obsidian after reading this.** The two plugin configs (QuickAdd, Spaced Repetition) were edited on disk while Obsidian was closed. They load on next launch. A timestamped backup of the QuickAdd config sits next to it (`data.json.bak-…`) if you need to revert.

---

## B. Two deliberate divergences from the plan (not mistakes)

1. **Field name is `reading_status` (underscore), not `reading-status` (hyphen).** Dataview's query language parses a bare hyphen as subtraction — `reading-status` in a `WHERE`/`SORT`/`GROUP BY` is read as *`reading` minus `status`* and silently matches nothing, which would leave every dashboard in §6 empty (the exact failure the plan warns about). Underscore is a valid identifier and resolves correctly. Same design intent, no collision with the vault's `status`. **Use `reading_status` everywhere** — template, Web Clipper, and when you flip the value by hand. The funnel query also uses Dataview's `key` (the documented post-`GROUP BY` accessor) rather than the field name.
2. **QuickAdd "New Source" auto-names the file `Untitled Source <timestamp>`, then the template renames it.** QuickAdd only shows its own filename prompt when the format contains `{{VALUE}}`; a static format suppresses it. This avoids a redundant double-prompt — you get just the template's two prompts (**Source title**, then **URL or PDF path**), and the title you type becomes the filename. Net behaviour matches the plan's intent ("leave Templater's prompt to rename").

---

## C. Manual — Web Clipper (Step 4, browser extension, can't be set from the vault)

In the **Obsidian Web Clipper** extension → create/edit a template so clipped articles land already speaking the schema:

- **Output folder:** `Inbox/`
- **Note content:** full article (Markdown); in-browser highlights arrive as `> blockquotes`.
- **Properties (frontmatter)** — set exactly (note the **underscore**):

```yaml
Type: Source
Parent: "[[Inbox/00000|Link]]"
reading_status: inbox
source: {{url}}
author: {{author}}
tags: []
```

- **Delete** the clipper's default `category: Clippings` property (classification is deferred to the Idea note).
- Leave `Area` / `Category` / `Subcategory` unset.

> **Mobile boundary:** the clipper is desktop-first. On Android, Chrome/Brave block extensions (use Firefox/Kiwi, or share → Firefox → clip); on iOS it's Safari-only with occasionally flaky metadata. Desktop-only capture is fine for v1.

---

## D. Manual — PDF capture flow (Step 5, a habit, nothing to install)

`pdf-plus` is already enabled. PDFs are **not** converted to Markdown — the PDF is the record, a sidecar Source note is its reading index.

1. Drop the PDF into the vault (e.g. `Inbox/`).
2. Run **"New Source"** to create its companion note; set `source:` to the PDF's vault path.
3. Highlight in the PDF with pdf-plus; paste the resulting `[[paper.pdf#page=3&selection=…]]` backlinks into the note's `## Highlights`.

> **Sync caution:** the PDF binary adds payload to `system3-relay` + `obsidian-git`. Annotate a given paper from **one device at a time** to avoid `.sync-conflict` artefacts (which `Index.md` already surfaces).

---

## E. The digest habit (where learning happens)

A source is **done** only when it has produced a `Type: Idea` note, in your own words, that links back to it. The flow:

1. **Read & filter** — mark keepers with `==highlights==` (web) or pdf-plus colours (PDF). Set `reading_status: reading` when you start.
2. **Annotate** — write the thought each highlight triggers; that marginal note is the high-value move.
3. **Cluster** by concept, *not* by the source's chapter order.
4. **Write the Idea note(s)** — existing `IdeaTemplate`, concept-oriented title, one idea per note, in your own words. Add `source-notes:: [[<the Source>]]`; ideally link one related Idea in a **different Area**.
5. **Close the loop** — add the Idea's wikilink to the Source's `ideas-extracted`, set `reading_status: integrated`. It drops off the queue.

Not every source earns an Idea note. Honest abandonment in v1 = delete the capture, guilt-free. The rule is "every highlight you **keep** owes an Idea note," never "every article you open must produce one." Don't let an AI summariser do the generative step — that step is the point.

---

## F. Acceptance test (run once, after reopening Obsidian)

v1 is correctly installed when all seven pass:

1. **Capture (web):** clip an article → a note appears in `Inbox/` with `Type: Source`, `reading_status: inbox`, `Parent: [[Inbox/00000|Link]]`, `source:` populated.
2. **Capture (manual/PDF):** run **"New Source"** → you're prompted for **Source title** then **URL or PDF path**; a stamped Source note is created in `Inbox/`.
3. **Queue shows it:** open `Index.md` → the new Source appears in the **Reading queue**; the **Inbox count** incremented by the right number.
4. **Advance:** set the Source's `reading_status: reading` → it leaves the queue and moves to the `reading` row of the **Digestion funnel**.
5. **Integrate:** write a `Type: Idea` note, link it back, add it to `ideas-extracted`, set `reading_status: integrated` → the Source moves to the `integrated` row; the Idea note exists with a backlink.
6. **Audit clean:** confirm the Source does **not** appear in `Index.md` → *Files without parent links* (Parent is set), and no new `.sync-conflict` files appeared.
7. **Highlight integrity:** confirm a `==highlighted==` phrase did **not** become a spaced-repetition card (Step 6 toggle worked).

If all seven pass, v1 is done. Use it for a few weeks before touching the §8 deferral table in the plan.

---

## G. Quick reference

| Thing | Value |
|---|---|
| New `Type` value | `Source` |
| New field | `reading_status` ∈ {`inbox`, `reading`, `integrated`} |
| New folder | `Inbox/` (flat) + `Inbox/00000.md` anchor |
| New template | `Templates/SourceTemplate.md` |
| Capture (web) | Web Clipper → `Inbox/`, stamps `Type: Source` + `reading_status: inbox` |
| Capture (PDF/manual) | QuickAdd **"New Source"** + pdf-plus highlights |
| Highlight marker | `==process me==` (+ `> blockquote` from clipper, `[[file.pdf#…]]` from pdf-plus) |
| "Done" = | a `Type: Idea` note links back → flip Source to `integrated` |
| Settings toggle | `==`/`**` disabled as SR cloze markers; `{{ }}` enabled |
| Dashboards | Reading queue + Inbox count + Digestion funnel on `Index.md` |
| Best single metric | is the Idea folder growing? |

---
*Implements Phase 1 of [[Information Intake — v1 Implementation Plan]]. Source architecture: [[SYSTEM]].*
