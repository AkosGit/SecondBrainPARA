Here's the v1 loop plus two concrete, copy-pasteable walkthroughs — one for a paper (PDF), one for a web article (link).

## v1 — the whole process in one loop

```
 CAPTURE ─────────► READ + HIGHLIGHT ─────────► EXTRACT ──────────► DONE
 (Type: Source       (== / pdf-plus marks       (write 1 Type: Idea   (reading-status:
  reading-status:      = "process me"            note in your words,    integrated, because
  inbox, in Inbox/)    pointers, no obligation)  backlink the Source)   ideas-extracted ≠ empty)
```

Six moving parts, nothing more:
1. **One folder:** `Inbox/`.
2. **One new Type:** `Source` (web article or paper). Web → Obsidian Web Clipper; PDF → drop the file in + a companion Source note via a QuickAdd/Templater "New Source" command.
3. **One field:** `reading-status`, starting at `inbox`. v1 only needs two real values: `inbox` and `integrated` (add `reading` only if you actually track in-progress reading).
4. **One habit:** when you read a Source, highlight freely as a *pointer*, then write **at least one `Type: Idea` note in your own words** and add its wikilink to the Source's `ideas-extracted` list. That flip — `ideas-extracted` going non-empty — is what makes it `integrated`. Highlighting alone never counts as done.
5. **One dashboard signal:** a single **count** of `inbox` items on `Index.md` (a number, not an alarm). If it embarrasses you, you process.
6. **One setting:** in obsidian-spaced-repetition, disable `==`/`**` as cloze markers so your highlights don't accidentally become flashcards. (Only matters once you add SR in v2.)

Everything else from the report — the 5-state lifecycle, WIP/staleness alarms, `ideas-extracted` analytics, spaced repetition, `maturity` field, MOCs — is **v2**, added only when a specific pain appears.

---

## Example A — a paper (PDF)

Say you're reading **Dunlosky et al. 2013, "Improving Students' Learning With Effective Learning Techniques."**

**Step 1 — Capture.** Drop `dunlosky-2013-learning-techniques.pdf` into `Inbox/` (or an `Attachments/` folder) and fire your "New Source" command, which creates the companion note `Inbox/Dunlosky 2013 — Effective Learning Techniques.md`:

```yaml
---
Type: Source
Parent:
Area:
Category:
Subcategory:
reading-status: inbox
source: "[[dunlosky-2013-learning-techniques.pdf]]"
author: Dunlosky, Rawson, Marsh, Nathan, Willingham
ideas-extracted: []
tags: []
---

# Dunlosky 2013 — Effective Learning Techniques

## Highlights

## Digest (Phase 2 — owes an Idea note)
- [ ] Cluster highlights by concept
- [ ] Write Idea note(s) in own words, backlink here, add to `ideas-extracted`
- [ ] Flip `reading-status` to `integrated`
```

Note `Parent`/`Area`/`Category` are left **blank** — you classify at the digest step, not at capture.

**Step 2 — Read + highlight (in the PDF, via pdf-plus).** Select a passage in the PDF viewer, click a color in the pdf-plus palette, and paste the backlink into the `## Highlights` section. Each one looks like this and renders as a colored highlight back in the PDF:

```markdown
## Highlights
> [!quote|yellow] [[dunlosky-2013-learning-techniques.pdf#page=4&selection=...&color=yellow|p.4]]
> > Highlighting and underlining... did not consistently boost performance.
> Marginal note: so my own highlighting habit is the weakest of the ten — confirms I should not stop here.
```

That marginal note is the cheap "constructive" move that already starts the learning. The highlight itself is just a **pointer** — it owes Step 3.

**Step 3 — Extract one Idea note.** Cluster the highlights by concept and write a `Type: Idea` note, titled as a *claim*, not as the source. New file `AREAS/Learning/Retrieval and spacing beat highlighting.md`:

```yaml
---
Type: Idea
Parent: "[[AREAS/Learning/00000|Learning]]"
Area: Learning
Category: Learning science
Subcategory: study techniques
source-notes:
  - "[[Dunlosky 2013 — Effective Learning Techniques]]"
tags: []
---

# Retrieval and spacing beat highlighting

Highlighting is the *lowest*-utility of ten studied techniques (d≈0.44 — weak,
not zero); practice testing and distributed practice are the *highest*. So in my
own reading, a highlight should only ever be a marker for where to do the real
work — rewriting in my own words.

Connects to: [[Generative encoding beats transcription]] · [[Forgetting can aid learning]]
```

**Step 4 — Close the loop.** Add that Idea's wikilink back into the Source's `ideas-extracted`, and set the status:

```yaml
reading-status: integrated
ideas-extracted:
  - "[[Retrieval and spacing beat highlighting]]"
```

The paper is now `integrated` — the only state that counts as "learned." The PDF binary stays in the vault and syncs with everything else (annotate a given PDF from one device at a time to avoid sync conflicts).

---

## Example B — a web article (link)

Say you're reading **Andy Matuschak's "Evergreen notes should be concept-oriented."**

**Step 1 — Capture (Web Clipper).** Click the Obsidian Web Clipper. With a per-site template configured once to write your schema, it lands `Inbox/Evergreen notes should be concept-oriented.md` already speaking your vocabulary — delete the clipper's default `category: Clippings` property; your `Category`/`Subcategory` get set later at digest:

```yaml
---
Type: Source
reading-status: inbox
Parent:
source: "https://notes.andymatuschak.org/Evergreen_notes_should_be_concept-oriented"
author: Andy Matuschak
published: 2020
ideas-extracted: []
tags: []
---

# Evergreen notes should be concept-oriented

> Evergreen notes are factored and titled so that they're useful across
> multiple contexts, from multiple perspectives...   ← (Web Clipper highlights
> arrive as blockquotes like this)

[full clipped article body follows]
```

The difference from the PDF: a web article **is** its Source note (Markdown all the way down). Web Clipper highlights come in as `> blockquotes`; the marks *you* add by hand are `==highlights==`.

**Step 2 — Read + highlight.** As you read in Obsidian, wrap the passages that resonate in `==...==`:

```markdown
...the question shifts from ==“under which topic should I store this note?”
to “in which contexts will I want to stumble upon it again?”==
```

`==` means "process me." Nothing is learned yet — it's a promise.

**Step 3 — Extract one Idea note.** `AREAS/PKM/Title notes as the idea, not the source.md`:

```yaml
---
Type: Idea
Parent: "[[AREAS/PKM/00000|PKM]]"
Area: PKM
Category: note-taking
Subcategory: zettelkasten
source-notes:
  - "[[Evergreen notes should be concept-oriented]]"
tags: []
---

# Title notes by the idea, not the source

If I name a note for its claim ("X causes Y") instead of its source ("Notes on
Smith 2020"), then the next time I meet that idea I *update the existing note*
instead of starting a new one — which forces synthesis and surfaces tension
between sources. This is why my Source→Idea step must produce concept-titled notes.

Connects to: [[Cross-domain links generate the most surprising ideas]]
```

**Step 4 — Close the loop.** On the Source note:

```yaml
reading-status: integrated
ideas-extracted:
  - "[[Title notes by the idea, not the source]]"
```

---

## The v1 dashboard (drop on `Index.md`)

One number — what's still owed a read:

````markdown
```dataview
TABLE WITHOUT ID length(rows.file.link) AS "📥 Waiting to read"
FROM "Inbox"
WHERE Type = "Source" AND reading-status AND reading-status = "inbox"
GROUP BY true
```
````

And the queue itself, oldest-first so it drains FIFO:

````markdown
```dataview
TABLE WITHOUT ID file.link AS "Source", source AS "URL",
  (date(today) - file.ctime).days + " d" AS "Age"
FROM "Inbox"
WHERE Type = "Source" AND reading-status AND reading-status = "inbox"
SORT file.ctime ASC
```
````

**The core difference between the two source types:** a paper *has* a Source note (a Markdown sidecar pointing at the PDF, highlights live as pdf-plus backlinks); a web article *is* its Source note (Markdown, highlights live as `==marks==` / blockquotes). Both carry identical frontmatter, so the same dashboard treats them uniformly — and both reach "done" the same way: an Idea note in your own words, backlinked.

Want me to scaffold this for real in your SecondBrainPARA vault — create the `Type: Source` Templater template, the `Inbox/` folder + its `00000` anchor, the QuickAdd "New Source" command, and these two queries on `Index.md`?