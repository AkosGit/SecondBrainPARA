# An Information-Intake System for Your SecondBrainPARA Vault

## Executive Summary — the answer, first

Your instinct is right in *shape* and stalls at exactly the step you emphasized. Capture an article or paper, park it in a read-it-later state, digest it later, highlight what matters, learn from it — that sequence is sound, except that highlighting is where almost every reading system quietly dies. The move you framed as the place where learning happens is, in the learning-science literature, the weakest technique studied — not empty, but being treated as the *finish* rather than a filter. So the single rule this whole design hangs on is a reframing: a source is not "done" when you have read and highlighted it. It is done only when it has produced a new note, in your own words, that links back to it. Highlighting is a cheap, legitimate filter; it is a worthless terminus.

In your vault's own vocabulary, the system is small. A captured article or paper becomes a `Type: Source` note — one new value on your existing `Type` enum — carrying a `reading-status` field, named `reading-status` rather than bare `status` because your vault already uses `status` for the draft/review/evergreen lifecycle. Web articles arrive as durable Markdown via the Obsidian Web Clipper; papers stay as PDFs annotated in place by pdf-plus, each with a companion Source note [[obsidian-web-clipper]] [[obsidian-pdf-plusreadmemd-at-main-ryotaushioobsidian-pdf-plus-github]]. When you digest a Source, you extract at least one `Type: Idea` note — your existing enum value, your own atomic claim — and add its wikilink to the Source's `ideas-extracted` list. The instant that list is non-empty, the Source is `integrated`. That is the knowledge-creation event, and it is the one thing your dashboards measure.

Two cautions are load-bearing. First, the largest risk in this project is not that you miss a good article — it is that you build a beautiful intake pipeline and mistake tending it for learning [[i-built-a-second-brain-in-notion-and-obsidian-it-was-a-productivity-trap-make-te]]. So the design is *opinionated about the rule and restrained about the machinery*: build a genuinely minimal version first, and grow each richer feature only when a specific, felt pain forces it. Second, do not let any external tool — or any AI summarizer — perform the generative step for you; that step is the entire point. The minimal version is one folder, one new `Type`, one `reading-status` field, the Web Clipper, the one-Idea-per-source habit, and one count on a dashboard. The rest of this report argues that spine and hands you the schema tables, the Templater snippet, and the Dataview queries that make the queue drain itself.

## 1. Design Principles: Mapping Knowledge-Management Practice onto Your PARA Vault

Your vault already encodes a worldview the knowledge-management field spent two decades arriving at independently: organize information by *actionability*, not by subject. PARA (Projects, Areas, Resources, Archive) files things by where they are going — Projects end, Areas are ongoing responsibilities, Resources are reference material you might use someday, everything inactive is archived [[the-para-method-the-simple-system-for-organizing-your-digital-life-in-seconds]]. An intake system that respects this must answer one question for every incoming article and paper — *what is this for, and what will I make from it?* — and must answer it at the moment you actually know, which is during the digest, not at capture. Four principles follow.

### A. The queue is a conveyor belt, not a shelf

Because PARA files by destination, an intake pipeline that honors it does not build a giant topical "Articles" shelf; it builds a *staging lane* whose only job is to feed your actionable Areas and Projects. The read-it-later inbox is not a destination — it is a conveyor belt, and a conveyor belt that never moves is just a pile. Every item that enters the lane is expected to leave it, either by producing an idea or by being honestly dropped — which is what turns "save it for later" from a hoarding reflex into a workflow. The inbox is an asset only when coupled to a forcing function (Section 3 builds that function).

### B. The generative act is the unit of completion — and two independent literatures say so

This is the thesis, and it resolves the deepest disagreement in the field. On one side, Tiago Forte's Progressive Summarization treats layered highlighting as a legitimate, low-friction way to surface what matters — raw note, then bold, then highlight, then own-words summary, then remix [[progressive-summarization-a-practical-technique-for-designing-discoverable-notes]]. On the other, cognitive psychology is decisive that *marking text as a terminal act* contributes little to durable learning. Dunlosky's synthesis is blunt: highlighting "has been shown to have failed to help students of all sorts," and students "who highlighted while reading performed worse on tests of comprehension wherein they needed to make inferences that required connecting different ideas across the text" — because attention spent marking individual concepts is attention not spent on the connections between them [[strengthening-the-student-toolbox]].

What durably builds knowledge is a *generative* act — one that forces you to select, organize, and integrate new material with what you already know. Richard Mayer's framework names eight such strategies (summarizing, mapping, drawing, imagining, self-testing, self-explaining, teaching, enacting) [[eight-ways-to-promote-generative-learning-educational-psychology-review-springer]], and Chi and Wylie's ICAP framework — Interactive, Constructive, Active, Passive, in descending order of engagement — explains why: learning rises as engagement climbs that ladder, and *constructive* behavior (generating content not in the source, such as writing a connection or a question while reading) beats merely *active* behavior like highlighting or copying [[eric-ej1044018-the-icap-framework-linking-cognitive-engagement-to-active-learnin]].

The practical consequence is liberating: the generative act need not be a separate writing session. A marginal "but this contradicts X," written while reading, already counts. The Idea note simply makes that constructive act durable and queryable instead of trapped in a margin.

The reason to weight this rule so heavily is that it arrives by two unrelated roads. The learning-science road says generation beats marking. The behavioral road — Christian Tietze's collector's fallacy — says acquiring information only *feels* like knowing it: bookmarking fires a Skinnerian reward (the stack grows) that is decoupled from learning, and "if we read without taking notes, our knowledge increases for a short time only" [[the-collectors-fallacy-zettelkasten-method]]. When two literatures with nothing in common converge on the same constraint, treat it as the spine of the design. This one does.

### C. Highlighting is the weakest technique that still beats baseline — a filter, not a finish

It would be an overstatement to call highlighting inert. The most precise quantification in the evidence base complicates the easy dismissal: Donoghue and Hattie's 2021 meta-analysis (242 studies, 1,619 effect sizes, 169,179 participants) places underlining and summarization at **d=0.44** — the *lowest* of ten techniques, but still *above* Hattie's benchmark average of 0.40 drawn from over 1,200 meta-analyses [[frontiers-a-meta-analysis-of-ten-learning-techniques]]. Practice testing (d=0.74) and distributed practice (d=0.85) sit at the top; highlighting sits at the bottom but not at zero. So the honest rule is "highlighting is the weakest tool in the box, not an empty box" — a good first-pass filter and a near-useless finish line. The design demotes it to a *pointer* where a generative act is owed, not because it does nothing, but because the real gains live one step past it.

Forte's own method gets the triage logic right, even as the design departs from it. He applies his layers to a shrinking fraction of what he reads — saving notes on roughly 50% of sources, bolding ~25%, highlighting ~20%, summarizing ~5%, remixing under 1% — and warns that over-applying them "destroys the system's value" [[progressive-summarization-iii-guidelines-and-principles-forte-labs]]. The top layers (capture, highlight) are triage; the bottom layers (own-words summary, remix) are the generative work the learning science endorses.

Here is the honest departure: Forte expressly rejects the funnel reading — "this isn't a funnel, where the more notes reach the bottom… the better," and "more summarization is not better… leave it alone" — and relies on a *future project* to resurface the right highlights when you need them [[progressive-summarization-iii-guidelines-and-principles-forte-labs]]. The design adds a forcing function he deliberately omits: rather than waiting for serendipity, it demands a generative act on the highlights you choose to *keep*. That is justified because your goal is to learn and generate ideas now, not to maintain a discoverable archive for a project that may never come.

### D. Restraint: the system itself is the highest-probability failure mode

The fourth principle governs the other three, keeping an opinionated thesis from metastasizing into an over-engineered rig. The most insistent failure in the practitioner literature is not under-capturing — it is the elaborate system becoming the procrastination. A writer who built a "second brain" across Notion and Obsidian concluded it was "a productivity trap": configuring dashboards delivered "mini dopamine hits" of "geeky procrastination" that coexisted with no actual output [[i-built-a-second-brain-in-notion-and-obsidian-it-was-a-productivity-trap-make-te]]. It rhymes with the most adversarial thread in the Zettelkasten community: a long-time practitioner abandoned the method as "a coping mechanism," arguing the order it produces "is clearly just a pseudo-order and it only exists within the realm of whatever tool you are using" [[zettelkasten-as-a-coping-mechanism-or-why-i-abdicate-the-zettelkasten-method-zet]]. Every status state, every Dataview alarm, every forcing function you add is a new surface on which that procrastination grows.

There is a genuine counterweight: a system that captures *and then processes* is valuable. The one workplace study in the evidence base found that digital hoarding positively predicts job performance (**β=0.397**, replicated at **β=0.461**) because hoarded material acts as a resource reservoir — but only through a dual path where the processed-resource benefit (thriving) outweighs the unprocessed-anxiety cost (burnout), and where anxious, completist hoarding actively *attenuates* the benefit [[exploring-the-effect-of-digital-hoarding-in-the-workplace-on-employee-work-perfo]].

Read the β values as direction, not measurement: the study is workplace social-media favoriting among 211 Chinese employees scored on *job performance*, not a personal reading pipeline scored on *learning*, so only the shape of the finding ports. It warns that capture pays off only when coupled to deliberate processing, and that the anxious, WIP-tracked, completist variant is the one that hurts. So build for the version of you who processes a little, not the version who curates a lot. That is why Section 8's blueprint is phased: a minimal version first, a richer one only after the minimal one proves it will not become another abandoned dashboard.

One disagreement to settle here so it does not haunt the implementation. The PKM (personal knowledge management) canon leans heavily on "the pen is mightier than the keyboard" — Mueller and Oppenheimer's 2014 finding that typing verbatim notes produces shallower learning than longhand. That specific result did not robustly replicate: a preregistered direct replication (N=142) found no effect on conceptual performance (Hedges' g=−0.13, opposite in direction to the original 0.34), and Dunlosky's own group later concluded that deciding which method is superior "seems premature," with group differences nearly vanishing once students studied their notes [[pdf-dont-ditch-the-laptop-just-yet-a-direct-replication-of-mueller-and-oppenheim]] [[how-much-mightier-is-the-pen-than-the-keyboard-for-note-taking-a-replication-and]].

Do not build the system on "rewrite by hand." The *mechanism* the famous study gestured at — generative encoding beats transcription — survives independently through self-explanation and the testing effect. Ground "rewrite in your own words" in that convergent evidence, not a single viral paper.

## 2. The Capture Layer: A Read-It-Later Inbox and Ingestion of Papers vs. Web Articles

The read-it-later queue is the part of your instinct that is both least controversial and most dangerous. It is a legitimate first step — "saving without processing creates backlog, not knowledge" is precisely the failure to engineer against [[i-solved-my-read-it-later-bookmarks-problem-with-a-reading-inbox-in-obsidian]]. This section gets things into the queue cheaply; Section 3 makes the queue drain.

### A. One flat inbox, zero sub-structure

Make a single folder — `Inbox/` (or `RESOURCES/Inbox/`) — and keep it deliberately structureless. The most battle-tested practitioner pattern is also the simplest: "the moment you add sub-folders or tags to this folder, you've created a filing system that needs more clicks instead of an inbox" [[i-solved-my-read-it-later-bookmarks-problem-with-a-reading-inbox-in-obsidian]]. The inbox is a triage tray, not an archive. Your vault's heavier classification machinery — the `Parent` wikilink to the folder's `00000` anchor note, the `Area`/`Category`/`Subcategory` fields copied from the sibling `00000` at creation and then frozen — gets applied at the *digest* step, not at capture. Capture must be a single action; classification is deferred until you actually know what the source is for.

### B. Web articles: the Obsidian Web Clipper, desktop-first

For HTML, use the Obsidian Web Clipper — a free, MIT-licensed extension maintained by the Obsidian team that saves the full article as durable Markdown into a chosen folder, extracts metadata (title, author, date, URL) into frontmatter, and stores any in-browser highlights as Markdown blockquotes [[obsidian-web-clipper]]. Its per-site templates can write *your* frontmatter one-to-one, so a clipped note lands already speaking your schema [[the-obsidian-web-clipper-fixes-everything-obsidian-rocks]]. Point its template at `Inbox/` and have it stamp `Type: Source` and `reading-status: inbox` automatically. The philosophy underneath it is the one that should govern the whole vault — "file over app": "in the fullness of time, the files you create are more important than the tools you use to create them" [[obsidian-web-clipper-steph-ango]].

The honest caveat is mobile, and you should name it as a boundary condition rather than wave it away. On Android, Chrome and Brave block extensions entirely, so the clipper does not run there without a workaround browser (Firefox, Kiwi) or a two-step "share to Firefox then clip" dance [[official-web-clipper-on-android-help-obsidian-forum]]. On iOS it runs in Safari only, and metadata extraction has been flaky on mobile-optimized sites. **The decision rule is concrete: if you capture mostly on desktop, native capture is strictly better on durability and zero lock-in. If you read mostly on a phone, the clipper's mobile friction is real, and the external seam in Section 8 is for you.**

### C. Papers (PDFs): pdf-plus, annotated in place, with a companion Source note

PDFs are a different substrate and should be ingested differently. Do not convert them to Markdown — keep the PDF as the source of record, store it in the vault, and create a sidecar `Type: Source` note as its reading index. pdf-plus is the right tool because of one design decision: it "transforms backlinks to PDF files into highlight annotations… it does not modify the PDF file itself," and the annotations live as *pure Markdown* — "you will not lose your annotations even if the plugin stops working, as long as Obsidian is alive… It will not leave behind a pile of unreadable JSON" [[obsidian-pdf-plusreadmemd-at-main-ryotaushioobsidian-pdf-plus-github]]. Every PDF highlight becomes a wiki-link of the form `[[paper.pdf#page=3&selection=...&color=yellow]]` that the viewer renders as a colored highlight, with the color encoded in the link itself. This is the PDF analogue of an `==inline highlight==`: a pointer that owes a generative act.

The two source types reach *parity* without identical mechanics. A web article *is* its Source note — Markdown all the way down, its clipped highlights arriving as `> blockquotes` and your own marks added as `==highlights==`. A paper *has* a Source note — a Markdown sidecar pointing at the PDF, with pdf-plus highlight backlinks. Both carry identical frontmatter (`Type`, `reading-status`, `ideas-extracted`), so every dashboard in Section 7 treats them uniformly; the only difference is where the highlight pointer physically lives. In the minimal build, resist engineering further "ingestion parity" — a `source-type` enum and parallel pipelines — until a query genuinely needs to branch on it. The asymmetry that actually matters is behavioral: the Barbell Method names it — skim cheaply first to decide whether a dense paper deserves slow processing, "concentrate our energy and attention on the really important and dense books," and invest "only a necessary minimum of attention into the low quality content" [[the-barbell-method-of-reading-zettelkasten-method]]. That is a reading habit, not a frontmatter key.

One sync-scope caveat follows from keeping PDFs in the vault: pdf-plus stores the PDF binary itself in the vault, so each paper adds payload to your system3-relay + obsidian-git sync across desktop and mobile — large papers are worth weighing against whether they need to live in the synced vault at all. The annotations are safe (they are plain-Markdown backlinks that merge cleanly under git), but editing the *same* PDF on two devices can surface in the "Sync conflicted files" report your `Index.md` already tracks, so annotate a given paper from one device at a time.

### D. QuickAdd as the friction-killer

To make capture genuinely one-action, drive new Source-note creation through QuickAdd, which already ships in your stack and is built precisely for "save interesting links for later reading" [[capture-quickadd]]. A QuickAdd Template choice can create a file from your template, in `Inbox/`, with format tokens like `{{DATE}}`, `{{VALUE}}`, and `{{VDATE}}` filled at creation, then hand off to Templater for the suggester-driven Category/Subcategory fields [[format-syntax-quickadd]] [[template-quickadd]]. Use the Web Clipper for the bulk of web capture (it is faster and pulls full content); use a single QuickAdd command — "New Source" — as the manual on-ramp for PDFs and for the "I want a Source note but not the whole article body" case. This is the same pattern your seven existing Templater templates already use; you are extending the family, not inventing a mechanism.

Modal Forms — also already installed — is the right tool when a manual Source capture wants a richer structured form than a linear Templater prompt chain: it renders a dialog (multiple authors, a source-type or Category/Subcategory dropdown) whose answers serialize straight into the §8.A Source frontmatter, mirroring the vault's existing health-log form pattern. Keep it v2/optional, reached for only when the linear suggester chain feels too thin — it should not contradict the minimal-build thesis.

## 3. The Status Model: A Reading Lifecycle Encoded in Frontmatter

This is the heart of the forcing function — and it is also where an ambitious design overreaches hardest, so the lifecycle is presented here as a *concept* in full, then committed to in a deliberately minimal form for your first build.

### A. The field name: `reading-status`, never bare `status`

Your vault already uses `status` for the note lifecycle (draft / review / evergreen) and on DailyNote frontmatter. A bare `status` for reading would collide and silently corrupt both query families. Use `reading-status` exclusively for the reading lifecycle. This is a small naming decision with a large blast radius; get it right at the start, and every Dataview query in Section 7 can filter on `reading-status` without ever catching a draft research note.

### B. The full lifecycle, as a concept

The complete lifecycle is five values on a single `reading-status` field; see it whole before deciding how much to build:

```
       capture            read           digest          extract Idea
  ───────────► inbox ──────────► reading ────────► processed ────────► integrated
                 │                  │                  │
                 └──────────────────┴──────────────────┴──────────► dropped
```

| State | Meaning | Entry condition |
|---|---|---|
| `inbox` | Captured, not yet opened — the collector's-fallacy danger zone | Set automatically at capture |
| `reading` | Actively reading / highlighting | You open it and start |
| `processed` | Read and highlighted; the generative act is not yet done | You finish reading; `==highlights==` exist |
| `integrated` | At least one `Type: Idea` note backlinks here | `ideas-extracted` is non-empty |
| `dropped` | Abandoned, honestly — an exit, not a failure | Manual |

Two design choices in this model are the whole thesis made checkable. First, the distinction between `processed` and `integrated` encodes the exact gap this report is about. `processed` is "I read it and marked it up" — the state most read-it-later systems treat as the finish line. `integrated` is "I made something from it." The most independently-corroborated finding in the evidence base is that these are not the same thing, and the status model makes the gap *visible*: every Source stuck at `processed` is a measured debt. Second, `integrated` is defined mechanically, not by feel: `reading-status: integrated` ⟺ `length(ideas-extracted) > 0`. You cannot fake "done" by toggling a field, because "done" *is* the existence of a backlinking Idea note, checkable against Dataview's always-available index [[metadata-on-pages-dataview]]. The `dropped` state matters too — Tietze's prescription is to "drain the stack," not to process everything [[the-collectors-fallacy-zettelkasten-method]], and "you don't have to save everything" [[i-solved-my-read-it-later-bookmarks-problem-with-a-reading-inbox-in-obsidian]]. A queue with zero drops is one you are lying to yourself about.

### C. What to actually build first: a minimal status set and a count

Here is where restraint governs. The full five-state lifecycle is the *concept* and the v2 target, not the v1 default, and the caution is empirical about how reading trackers actually get used. Every working community implementation tracks at most *three* states, and they are reading-progress states, not knowledge-creation states: one book tracker uses `shelf: toread | reading | read | stopped` [[how-i-track-books-and-reading-with-obsidian-aaron-young]]; another uses `Status: Unread | Reading | Read` [[bookshelf-and-reading-tracker-with-obsidian-by-iolar-saor-medium]]. Absent from every shipped system: an `integrated` state, a work-in-progress (WIP) cap, a staleness alarm. Read that absence carefully — these are *book-completion* trackers where "read = done" genuinely is the goal, so the omission is evidence they solve a different problem, not proof the state is worthless for a learning pipeline. The real case for v1 minimalism is the procrastination-surface argument: each extra state and alarm is marginal discipline rarely worth its marginal bookkeeping.

So the committed v1 is minimal: a small `reading-status` (start with `inbox` and `integrated`, or add `reading` if you genuinely track in-progress reading), the one-Idea-per-source habit, and a single *count* of inbox items — not a cap, not an alarm. Use the value `integrated` for "done," not a separate `done` token, so the v1 statuses are a strict subset of the full vocabulary and every copy-paste query in §7 works unchanged.

A count beats a cap because the alarm numbers (a cap around 10, a staleness threshold around 14 days) are *uncalibrated* to your real intake rate; they are guesses, and a dashboard that turns red on a guess trains you to dismiss it. A single honest number does the work without the apparatus: if it embarrasses you, you process; if not, no automated banner will change your behavior. The knowledge-creation event stays where it belongs — the existence of an Idea note, queryable directly — rather than mirrored into a status enum that can drift from reality. The `processed`→`integrated` distinction, the `ideas-extracted` analytics, and the staleness alarm all move to v2, each behind a named trigger (Section 8).

This minimal model still answers your literal request — "put them in a read-it-later status and later digest them" — but it adds the one thing the naive version lacks: a definition of *later* that has teeth. "Later" is not "someday"; it is "before that count gets embarrassing."

## 4. The Digest Workflow: Highlighting, Annotation, and Progressive Summarization

This is the heart of your question — "highlight what info is important, so I can learn" — and it is where the evidence cuts both ways, so the recipe is written as two views of one rule.

### A. Phase 1: highlight as filter, with the syntax that keeps it clean

While reading, mark what matters. There are two distinct highlight channels for web articles: in-browser selections you make while clipping arrive as Markdown blockquotes (`> quoted passage`), because the Web Clipper does *not* emit `==mark==` [[obsidian-web-clipper]]; the `==inline highlights==` you add by hand later in the Source note are what carry the "process me" meaning. In a PDF, use pdf-plus color highlights, which become `[[paper.pdf#page=N&selection=...&color=...]]` backlinks [[obsidian-pdf-plusreadmemd-at-main-ryotaushioobsidian-pdf-plus-github]]. This is Progressive Summarization's lower layers, used exactly as Forte intends — opportunistic compression so that next time you open the note, the best passages are scannable in seconds; Layer 3 highlighting can reduce a 373-word note to 60 words, 6–12 times faster to scan [[progressive-summarization-ii-examples-and-metaphors-forte-labs]]. Do this freely; it is cheap. But a highlight is a pointer that owes Phase 2 — it makes the note findable, not learned.

To make this physically implementable in your exact plugin stack, adopt one syntax convention and never break it:

| Marker | Meaning | Where |
|---|---|---|
| `==text==` (and `**bold**`) | Hand-typed Progressive-Summarization / "process me" pointer | Source note body |
| `> quoted passage` | Web Clipper in-browser highlight (auto-emitted; add `==` manually to flag it) | Web-article Source body |
| `[[file.pdf#…&color=…]]` | pdf-plus highlight backlink (the PDF pointer) | Source note body (PDF) |
| `{{the answer phrase}}` (curly) | Spaced-repetition cloze (a fill-in-the-blank card) — the whole braced span is the deletion; the plugin uses **bare** `{{…}}`, not Anki's `{{c1::…}}` numbering | Idea note only |
| `#review` | Whole-note spaced-repetition review (while reading) | Source note |
| `#flashcards/TOPIC` | Atomic spaced-repetition deck | Idea note only |

The one real collision is narrow: the spaced-repetition plugin also accepts `==` and `**` as cloze markers [[github-st3v3nmwobsidian-spaced-repetition-fight-the-forgetting-curve-by-reviewin]], so left at the defaults, your *hand-typed* highlights would silently become flashcards. (Clipper blockquotes and pdf-plus backlinks have zero overlap with cloze syntax, so the collision exists only for the `==`/`**` marks you type yourself.) Resolve it with a single settings toggle: **disable `==`/`**` as cloze markers and use `{{curly braces}}` exclusively.** Then `==` always means "process me" and `{{…}}` always means "test me," and the two never interfere. This one toggle is what makes the Section 4 → Section 6 pipeline implementable.

To push reading up the engagement ladder while you highlight, *annotate as you go*. pdf-plus's "Quote in callout" template wraps each highlight and your comment into a single backlink-pane item:

```
> [!{{calloutType}}|{{colorName}}] {{linkWithDisplay}}
> > {{text}}
```

Combined with the Better Search Views plugin and pdf-plus's filter-by-page, this turns the Obsidian backlinks pane into a Zotero-style annotation view where each highlight sits beside the note you wrote about it [[tips-backlinks-pane-ryotaushioobsidian-pdf-plus-wiki-github]]. That marginal comment is the ICAP "constructive" move [[eric-ej1044018-the-icap-framework-linking-cognitive-engagement-to-active-learnin]], and it is where self-explanation enters — a strategy with a meta-analytic effect of **g=0.55** across 64 reports, which outperforms even being *given* the explanation by an instructor, proof that the benefit is in the generating, not the content [[inducing-self-explanation-a-meta-analysis-educational-psychology-review-springer]]. The prompt is simple: as you read, ask "What does this mean? How does it relate to what I already know?"

### B. Phase 2: the mandatory generative act

Phase 2 is what flips a Source from `processed` to `integrated`, and it is the procedure Matuschak adapts from Tietze — collect, then process [[write-about-what-you-read-to-internalize-texts-deeply]]:

1. **Cluster.** Group your highlights and annotations into piles by *concept* — explicitly *not* by the source's chapter order. The clusters are "orthogonal to the content," forming around your purposes, not the author's [[how-to-process-reading-annotations-into-evergreen-notes]].
2. **Write a broad Idea note** capturing the big idea of a cluster, in your own words. If a cluster holds several big ideas, write several notes.
3. **Write finer Idea notes** for the nuanced atomic points inside the cluster.
4. **Connect.** Search your existing Idea notes for relevant prior thinking; link, merge, revise. This is where cross-domain links get made (Section 5).
5. **Revise and loop.** Return to the broad note and improve it given what the detailed notes revealed.

Each Idea note backlinks to its Source, and you add that Idea's wikilink to the Source's `ideas-extracted` list — the act that flips the Source to `integrated`. Highlighting and marginalia are insufficient on their own because "there isn't much pressure to synthesize, connect, or to get to the heart of things. And they don't add up to anything over time" [[write-about-what-you-read-to-internalize-texts-deeply]]. The Idea note is what makes reading *accrete*.

### C. Progressive summarization is the bridge, not the destination — and don't over-process

Layer the digest the way Forte does — capture, bold, highlight, then summarize in your own words — but treat the own-words summary as the hinge, because it is the first genuinely generative layer and the one the learning science endorses [[an-introduction-of-tiago-forte-his-ideas-and-some-comments-on-their-relations-to]]. A Source that has reached a one-paragraph own-words summary is ready to spawn Idea notes; a Source that has only `==highlights==` is still in debt. The mirror risk to under-processing is over-processing — applying every layer to every source, or demanding an Idea note from a throwaway link. Matuschak's caveat applies: "the best way to read is highly contextual," not every text warrants deep processing, and "the most effective readers and thinkers I know don't take notes when reading" [[write-about-what-you-read-to-internalize-texts-deeply]]. The `dropped` state is the escape hatch. The rule is "every highlight you *keep* owes an Idea note," not "every article you open must produce one." Your processing rate will, and should, look like a power law — Forte's own does.

## 5. From Highlights to Ideas: Atomic / Permanent Notes as First-Class Queryable Objects

The Idea note is the product of the system, and the design's job is to make it a first-class queryable object rather than a buried bullet. Your vault already has the machinery; we are pointing it at a clear two-tier mapping.

### A. Two tiers, mapped onto your existing enum

The Zettelkasten tradition debates whether to keep three note types (fleeting, literature, permanent) or collapse them. Ahrens, channelling Luhmann, prescribes three [[how-to-take-smart-notes-10-principles-to-revolutionize-your-note-taking-and-writ]]; the zettelkasten.de camp rejects the literature/permanent distinction as overhead [[create-zettel-from-reading-notes-according-to-the-principle-of-atomicity-zettelk]]. For an action-oriented PARA vault whose owner wants to learn and generate ideas — not run a publishing machine — the right answer is *two tiers*, and your existing enum supplies both:

| Tier | Role | Vault `Type` | Lifecycle |
|---|---|---|---|
| Literature note / reading index | The source in transit; its highlights, annotations, own-words summary | **`Source`** (new value) | `inbox → reading → processed → integrated` |
| Permanent / evergreen note | *Your own* atomic idea, in your words, reusable | **`Idea`** (existing) | concept-oriented, densely linked |

This is the one structural choice the report makes confidently. Add a *new* `Type: Source` for the captured reading note, distinct from your existing `Type: Resource`, and reuse `Type: Idea` for the extracted permanent note. A separate third "literature note" tier is rejected — it is just a Source whose `reading-status` is still `reading`.

The strongest case *for* keeping it separate is worth naming: Ahrens and Matuschak treat the literature note as structurally distinct because it is *source-scoped* and therefore only "weakly evergreen" — harder to build on, and a contamination risk if source-ordered material sits at the same tier as concept-oriented notes. The two-tier collapse defends against exactly that by keeping the source-scoped material inside the `Source` note (in its `## Highlights` vs `## Digest` sections) and reserving `Type: Idea` for concept-oriented notes — so the fleeting-vs-literature distinction is encoded by `reading-status` and section, not by a third Type. For a learning/ideation goal rather than a publishing one, that is enough.

Making source-vs-idea a *Type* difference and not merely a status flag is what lets the forcing function ask Dataview "show me Sources that have produced no Ideas," and keeps curated `Resource` material uncontaminated by raw clippings. Reserve `Resource` for its current meaning — already-digested, stable reference material — and let a Source *stay* a Source as a reading index rather than "graduating" into a Resource. That keeps the two roles legible: `Source` = "something I read and mined," `Resource` = "stable reference I curated."

The graceful-degradation fallback: if even one new `Type` value feels like too much schema, reuse `Type: Resource` and let `reading-status` carry the transit-vs-reference distinction (`Resource` + `reading-status: inbox` = a transit capture; `Resource` with no `reading-status` = a settled reference). You lose the clean "Sources that produced no Ideas" query and risk mixing raw clippings into curated material, but the spine survives untouched. The recommendation remains the new `Type: Source` — the marginal cost of one enum value is tiny in a vault that already harvests `Type` via Templater suggesters, and keeping `Resource` clean is worth it.

### B. What makes an Idea note "permanent," enforced at extract, never at capture

Graft three light Zettelkasten principles onto the existing `Type: Idea`, and enforce them only at the *extract* step:

- **Concept-oriented title.** Name the note by the idea, not the source — "Retrieval beats restudy for delayed retention," not "Notes on Roediger 2006." This is what lets the same note be *updated* when the concept recurs in a future source, forcing synthesis and surfacing tension between sources [[evergreen-notes-should-be-concept-oriented]]. As Ahrens reframes it, the question shifts from "under which topic do I store this?" to "in which context will I want to stumble on it again?"
- **Atomic — as an outcome, never a gate.** One idea per note, so it can be linked and recombined independently. But atomicity is the desired *outcome*, not an entry barrier. Sascha Fast's warning is sharp: treating atomicity as an input gate ("notes must be atomic before they enter") produces anxiety and underdeveloped "bare-claim prompt machines," and pushes the real thinking *outside* the system [[create-zettel-from-reading-notes-according-to-the-principle-of-atomicity-zettelk]]. Source notes are allowed to be messy; the discipline of fitting a refined note into the existing web is itself where synthesis happens.
- **Densely linked, especially across Areas.** At least one backlink to its Source, plus links to related Idea notes — and the cross-domain links are the high-value ones (see C).

One optional v2 addition fits the "learn and get new ideas over time" goal: the digital-garden maturity model — `seedling → budding → evergreen` — is how practitioners track an idea's development, alongside Gwern-style epistemic-status metadata [[growing-the-evergreens]] [[a-brief-history-ethos-of-the-digital-garden]]. An optional `maturity` field on `Type: Idea`, left off the minimal v1, lets a Dataview query surface seedling/thin ideas as the next candidates for the cross-domain linking this report endorses — the high-value generative move is returning to under-developed ideas and growing them. Keep it optional until the Idea folder is large enough that "which ideas are still thin?" becomes a real question.

Each Idea note inherits classification the way every other note in your vault does: a `Parent` wikilink to the relevant `00000` anchor, with `Area`/`Category`/`Subcategory` copied at creation and then frozen, per your denormalised-inheritance contract. An idea extracted while working on a Project links into that Project; an idea with no current home links into the relevant Area. The Source records *where it came from*; the Idea note is filed by *where you'll want to stumble on it again* — which is exactly PARA's logic [[the-para-method-the-simple-system-for-organizing-your-digital-life-in-seconds]].

### C. Why a queryable Type matters, and why linking is the idea engine

Because the Idea note carries `Type: Idea`, it is a first-class row in your vault database. Dataview auto-indexes implicit fields on every note — `file.inlinks`, `file.outlinks`, `file.tags`, `file.ctime` — without any manual metadata [[metadata-on-pages-dataview]]. So you can ask:

- Which Idea notes have only one source (thin ideas, ripe for a second)?
- Which are most linked-to (your load-bearing concepts)?
- Which Areas hold the most Idea notes (where your thinking is densest)?

None of that is possible if ideas hide as bullets inside Source notes.

There is a second, finer layer for the literal "highlight what's important" ask: an *individual* highlight can itself be queryable, not just countable. Dataview indexes every list item as `file.lists`, and a list item can carry an inline field in bracket form — `- ==key passage== [theme:: spaced-repetition]` [[adding-metadata-dataview]] [[metadata-on-pages-dataview]]. A single query over `file.lists` then rolls up every highlight tagged `[theme:: …]` or `[importance:: high]` across all Sources, so a highlight becomes a first-class object with its own attributes rather than a number in a funnel. Treat this as a v2 refinement — per-highlight tagging adds friction — but it is the corpus-supported way to make the highlights themselves, not only the Idea notes, query targets.

Your stated goal — "get new ideas" — is the real payoff, and the linking layer is where it lives. Constructing concept maps (building the links yourself) produces a large learning effect — **g=0.72**, substantially larger than studying pre-made maps (g=0.43), across 142 effect sizes and ~11,800 participants, and across both STEM and non-STEM domains [[studying-and-constructing-concept-maps-a-meta-analysis-educational-psychology-re]]. The active construction of connections, not passive viewing, is the mechanism — which is exactly what densely linking your Idea notes does.

A second literature reinforces it: a meta-analysis of 525 correlations from 79 studies (12,846 participants) found that semantic memory — "the ability to strategically retrieve information from long-term memory" — drives the memory-creativity relationship, and that highly creative individuals have *denser, more connected, less modular* semantic networks [[memory-and-creativity-a-meta-analytic-examination-of-the-relationship-between-me]]. A richly linked Idea graph is, mechanically, a denser semantic network.

There is a contrarian nuance worth baking in: semantic richness has costs as well as benefits, and excessive association to one cue can cause functional fixedness [[semantic-memory-and-creativity-the-costs-and-benefits-of-semantic-memory-structu]]. The design implication is sharp: **prefer cross-domain links over within-domain depth.** When you connect an Idea note, link it to an Idea in a *different* Area, because "the non-apparent connections are generally more beneficial to creative thinking than the obvious ones as they generate greater surprise" [[evergreen-notes-should-be-concept-oriented]]. And do *not* stand up the slip-box ceremony to get the payoff. You need no Folgezettel (sequential note-ID) scheme, no separate Zettelkasten folder, no atomicity gate; the existing `[[wikilink]]` and `Parent` machinery is enough, and density grows opportunistically.

Do you need a Zettelkasten at all? For your goal, no. Bob Doto is unambiguous — "a Zettelkasten is first and foremost a writing tool," and "if your interest is primarily in 'making connections' or examining ideas, then you can happily set aside both of these" [[a-zettelkasten-is-not-for-making-connections-bob-doto]]. Doto's sharper edge: he treats connection-for-its-own-sake as the *misconception* and roots insight in writing output, not linking per se.

So take the concept-map evidence as licensing linking as a learning act — but for *idea generation*, the payoff surfaces when you periodically draft *from* a cluster of linked notes, not merely accumulate links. The famous "after 1,000 notes, magic happens" payoff accrues over years of full-time use; designing your week around a benefit that arrives in 2030 is its own kind of procrastination. The light-linking layer gives you the evidence-backed part of the promise at a fraction of the cost.

## 6. Reinforcing Learning: Spaced Repetition and Review Loops

Spaced repetition — reviewing material at expanding intervals to fight the forgetting curve — is the durability layer: optional, high-value, easy to misuse, and squarely a v2 feature. The committed scope: apply it only to a curated subset of Idea notes, never to raw Sources, and never to everything. The plugin is already installed, so the tooling cost is zero; the only cost is authoring discipline, which you opt into per idea.

### A. Why it earns its keep, and the hard constraint that makes it work

Distributed practice (**d=0.85**) and practice testing (**d=0.74**) are the two highest-utility learning strategies in the entire literature [[frontiers-a-meta-analysis-of-ten-learning-techniques]] [[strengthening-the-student-toolbox]], so retrieval-over-time is not snake oil. And the benefit is not merely rote: Karpicke and Blunt showed retrieval practice beat elaborative concept-mapping on *every* measure — recall, inference, short answer, even concept-map construction itself — demonstrating that retrieval builds *conceptual* knowledge [[karpicke-j-d-blunt-j-r-2011-retrieval-practice-produces-more-learning-than-elabo]].

This does not contradict §5's use of concept-map construction (g=0.72) as the idea engine: Karpicke ranks the two for *retention* of given material, where retrieval wins, whereas §5 uses linking as the *generative* act of building connections — so the design uses linking for ideation and retrieval for durability rather than choosing between them. And the benefit transfers: Pan and Rickard's meta-analysis found a grand mean transfer effect of d=0.40, rising to d=0.58 with response congruency, with a further d=0.23 boost from *elaborated* retrieval practice [[pan-s-c-rickard-t-c-2018-transfer-of-test-enhanced-learning-meta-analytic-review]].

That d=0.23 boost names the constraint most people miss: the transfer benefit appears *only* when prompts are elaborative and inference-level. It comes specifically from "broad encoding methods… instructed to think of all related information, to construct a detailed explanation, multiple questions involving higher-order Bloom's taxonomy levels" [[pan-s-c-rickard-t-c-2018-transfer-of-test-enhanced-learning-meta-analytic-review]]. Factual prompts give you retention with near-zero higher-order transfer. So write prompts that interrogate the idea — "how does this connect to X?", "when does this fail?", "why is this true?" — not prompts that quiz a definition.

Matuschak's prompt-writing guide is the manual: prompts should be focused, precise, *consistent* (the same answer every time, to avoid retrieval-induced forgetting), tractable, and effortful, and "prompt design is task design" — each prompt is "a recurring task you are giving your future self" [[how-to-write-good-prompts-using-spaced-repetition-to-create-understanding]]. His six conceptual lenses (attributes, similarities/differences, parts/wholes, causes/effects, significance/implications) plus "salience prompts" designed to keep an idea live in awareness are the right templates, and the cost is genuinely low: "an easy prompt will consume 10–30 seconds across the entire first year of practice."

### B. The honest counterweight, and the one trap to avoid

There is a real cost, and the case for *not* making spaced repetition mandatory rests on it. Good prompts are hard to write, and bad prompts fail silently — you keep reviewing a card that teaches nothing and never find out [[how-to-write-good-prompts-using-spaced-repetition-to-create-understanding]]. Multiplied across an intake pipeline, that becomes a second backlog — a *review* backlog — on top of the reading backlog you were trying to drain. And forgetting is not only a cost to fight: Forte argues "the faster you forget, the faster you learn," because offloading frees bandwidth and most of what you forget was noise — research on problem-solving even found performance improving as knowledge was forgotten, right up to the ~90% mark [[progressive-summarization-v-the-faster-you-forget-the-faster-you-learn-forte-lab]].

For many readers the most valuable review loop is not spaced repetition at all — it is the periodic re-encounter PARA already gives you, surfacing a note when an active Project pulls it back into relevance. Spaced repetition is the right tool for the handful of ideas you are actively failing to retain and genuinely want top-of-mind; it is the wrong default for everything.

In your stack, two review modes split cleanly by note type, and they are mutually exclusive per note: a note is in whole-note-review mode *or* card-review mode, not both. On Source notes, the `#review` tag puts the whole note into the plugin's Note Review Queue — useful while you are still reading [[github-st3v3nmwobsidian-spaced-repetition-fight-the-forgetting-curve-by-reviewin]]. On Idea notes, `{{curly}}` cloze cards under a `#flashcards/TOPIC` deck handle atomic retrieval after extraction; scheduling is stored invisibly in HTML comments so cards don't clutter the note, the plugin supports the FSRS or SM-2 algorithms, and review is single-key (0 = Again, 1 = Hard, 2 = Good, 3 = Easy) [[reviewing-cramming-obsidian-spaced-repetition]]. Do *not* review raw highlights — that is exactly the inert activity the design demotes.

That yields the one hard rule of the build-vs-buy seam: **do not run two review queues.** If you adopt Readwise Reader (Section 8), it ships its own spaced-repetition Daily Review over your raw highlights [[readwise-reader-review-2026-worth-it-speed-reading-lounge]]. Spaced re-exposure of highlights is a legitimate-if-weak retention mechanism — by the distributed-practice evidence above (d=0.85), re-seeing a passage beats never seeing it again. But it re-exposes you to the *author's* framing rather than the ideas you generated and rephrased, and running it *and* the vault's queue over Idea notes splits your learning loop across two artifact sets. Pick one, and keep it in the vault, over your own ideas.

## 7. The Dashboard Layer: Dataview Views for the Reading Queue and Digestion Progress

Dataview turns the status model from a convention you must remember into an ambient visual constraint you can't ignore. But heed the restraint principle here especially: every query is another surface for procrastination, and Dataview *silently drops* notes whose `WHERE` field is missing, so an inconsistent `reading-status` corrupts results invisibly. Build the minimum that makes the backlog visible. All queries below are copy-pasteable; tune the numbers to your real intake rate.

### A. v1 — the reading queue and a count

The whole v1 dashboard is two things: a queue of what to read next, and one honest number. The queue, oldest-first so it drains first-in-first-out rather than letting new captures bury old commitments:

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

`file.ctime` (file creation time) is implicit on every note, so nothing extra needs maintaining [[metadata-on-pages-dataview]]. The count — the v1 forcing function, a number not an alarm:

```dataview
TABLE WITHOUT ID length(rows.file.link) AS "Items waiting to be read"
FROM "Inbox"
WHERE Type = "Source" AND reading-status AND reading-status = "inbox"
GROUP BY true
```

Surface this on `Index.md` beside your existing "Files without parent links" query, so the backlog sits in the same field of view as the rest of your vault's health. A single honest number does the work Tietze's "drain the stack" intends [[the-collectors-fallacy-zettelkasten-method]]; the working community trackers confirm that plain status-filtered tables, not alarm systems, are what people actually maintain [[how-i-track-books-and-reading-with-obsidian-aaron-young]] [[bookshelf-and-reading-tracker-with-obsidian-by-iolar-saor-medium]].

### B. v1 — digestion progress (the funnel)

To watch the pile move rightward, group your Sources by state — the only metric that measures *learning*, not collecting:

```dataview
TABLE WITHOUT ID
  reading-status AS "Stage",
  length(rows.file.link) AS "Count"
FROM "Inbox" OR "RESOURCES"
WHERE Type = "Source" AND reading-status
GROUP BY reading-status
SORT length(rows.file.link) DESC
```

`GROUP BY` yields one row per status with a `rows` array you count via `length(rows.file.link)` [[data-commands-dataview]] — a one-glance funnel from `inbox` to `integrated`. The leading `WHERE reading-status AND …` guard is not optional: a `WHERE` on a missing field *silently drops* the note rather than erroring, so the guard is what keeps an un-stamped capture from vanishing from the count. The truest health check, though, costs no query at all: glance at your Idea folder. If it is growing, the system works.

### C. v2 — the staleness alarm and the unprocessed backlog

Add these only when "read vs not read" stops being enough to see where ideas are stalling. The staleness alarm surfaces items languishing too long:

```dataview
TABLE WITHOUT ID
  file.link AS "Stale source",
  (date(today) - file.ctime).days + " d old" AS "Languishing"
FROM "Inbox"
WHERE Type = "Source" AND reading-status
  AND (reading-status = "inbox" OR reading-status = "reading")
  AND file.ctime <= date(today) - dur(14 days)
SORT file.ctime ASC
```

The `dur(14 days)` window is a default to tune, and the date arithmetic is the documented pattern [[data-commands-dataview]]. Keying off implicit `file.ctime` (always a real date object) sidesteps the parsing trap of an explicit capture field: if you ever swap in a `date-captured` frontmatter field, store it as a `YYYY-MM-DD` ISO string or wrap it as `date(date-captured)`, since text dates do not support arithmetic. The unprocessed-backlog query is the thesis as a query — Sources read but producing nothing yet:

```dataview
TABLE WITHOUT ID
  file.link AS "Source",
  length(ideas-extracted) AS "Ideas made"
FROM "Inbox" OR "RESOURCES"
WHERE Type = "Source" AND reading-status AND reading-status = "processed"
  AND (!ideas-extracted OR length(ideas-extracted) = 0)
SORT file.mtime ASC
```

When `ideas-extracted` fills in, the Source self-promotes to `integrated` and falls off this list — the dashboard and the status model are two views of one rule. A visual reading shelf (cover thumbnails via `TABLE WITHOUT ID` and string-concatenated image URLs [[how-i-track-books-and-reading-with-obsidian-aaron-young]]) and a Kanban board are pure presentation — defer both to v2; pick whichever front-end you'll actually look at. The Kanban plugin is Markdown-backed — columns are Markdown headings, cards are list items — so it cross-indexes with Dataview rather than introducing a parallel store [[github-obsidian-communityobsidian-kanban-create-markdown-backed-kanban-boards-in]]. Because your vault's `Boards.md` already mines `file.tasks` vault-wide, the reading pipeline can ride that existing convention: rather than the board reading a `reading-status` frontmatter field directly, the natural integration is a dashboard note (or board) whose columns mirror the lifecycle states and that queries the `Inbox/` folder — reuse, not a second board system.

## 8. Implementation Blueprint: New Type Values, Templates, Frontmatter Keys, and Plugins

Everything below extends your existing contract; nothing replaces it, and every piece resolves to plain Markdown you control. The section delivers the concrete artifacts, then the phased build order so you know what to stand up first.

### A. Frontmatter schema — `Type: Source` (the captured reading note)

| Key | Type | Set by | Meaning |
|---|---|---|---|
| `Type` | enum | Template (fixed) | New value `Source` — captured reading note / literature index |
| `Parent` | wikilink | Templater suggester (deferred to digest) | Link to the owning folder's `00000` anchor |
| `Area` | text | Inherited from `00000` at digest | PARA Area, frozen at creation |
| `Category` | text | Templater suggester + "Other…" | Vault-global free vocabulary |
| `Subcategory` | text | Templater suggester + "Other…" | Vault-global free vocabulary |
| `reading-status` | enum | Template/clipper `inbox`; you advance | `inbox → reading → processed → integrated` / `dropped` |
| `source` | url | Web Clipper / QuickAdd | Original URL or PDF path |
| `author` | text/list | Web Clipper | One or many authors |
| `ideas-extracted` | list of wikilinks | You, at digest (v2) | Backlinks to `Type: Idea` notes; non-empty ⟺ `integrated` |
| `tags` | list | init `[]`; `#review` while reading | Vault tag contract |

Capture date needs no key — `file.ctime` is implicit on every note. A `source-type` enum (`web`/`pdf`) is optional and belongs in v2, only once a query needs to branch on it.

### B. Frontmatter schema — `Type: Idea` (the extracted permanent note)

| Key | Type | Set by | Meaning |
|---|---|---|---|
| `Type` | enum | Template (fixed) | Existing value `Idea` — your own atomic claim |
| `Parent` | wikilink | Templater suggester | Owning Area/Project `00000` anchor |
| `Area` / `Category` / `Subcategory` | text | Inherited, frozen | PARA classification |
| `source-notes` | list of wikilinks | You | Backlink(s) to the `Type: Source` note(s) this idea came from |
| `tags` | list | init `[]`; `#flashcards/TOPIC` if SR'd | Tag contract + spaced-repetition deck |

The reciprocal pair — `ideas-extracted` on the Source, `source-notes` on the Idea — is what makes "done" bidirectional and queryable from either end.

### C. Templater snippet for a new Source note

```javascript
<%*
// New Source note — drop in Inbox, defer classification to digest
const title = await tp.system.prompt("Source title");
const url   = await tp.system.prompt("URL or PDF path", "");
const cat   = await tp.system.suggester(
  ["Other…"].concat(tp.user.categories?.() ?? []),
  ["Other…"].concat(tp.user.categories?.() ?? [])
);
const category = cat === "Other…" ? await tp.system.prompt("New Category") : cat;
await tp.file.rename(title);
-%>
---
Type: Source
Parent:
Area:
Category: <% category %>
Subcategory:
reading-status: inbox
source: <% url %>
author:
ideas-extracted: []
tags: []
---

# <% title %>

## Highlights
<%* /* ==highlights== or pdf-plus backlinks land here while reading */ -%>

## Digest (Phase 2 — owes an Idea note)
- [ ] Cluster highlights by concept
- [ ] Write Idea note(s) in own words, backlink here, add to `ideas-extracted`
- [ ] Flip `reading-status` to `integrated`
```

The embedded checklist is itself a small forcing function — the Source note tells you what it owes — and the suggester + "Other…" escape hatch mirrors your existing Category/Subcategory pattern exactly [[capture-quickadd]].

The Web Clipper needs the matching artifact: a per-site template whose Properties write your schema directly so a clipped article lands already speaking it. Out of the box the clipper stamps `category: Clippings`, which you must remap [[the-obsidian-web-clipper-fixes-everything-obsidian-rocks]] [[obsidian-web-clipper]]. Configure the template's Properties to emit:

```yaml
Type: Source
reading-status: inbox
Parent:
source: {{url}}
author: {{author}}
published: {{published}}
tags: []
```

Delete the default `category: Clippings` property (your `Category`/`Subcategory` are set by suggester at the digest step, not at capture), point the template's output folder at `Inbox/`, and leave `Parent`/`Area` blank for the same reason. That block is the whole clipper-side mapping — nothing else needs configuring for a clip to enter the pipeline correctly.

### D. Build-vs-buy: native Obsidian vs Readwise vs Raindrop

| Dimension | Native (Web Clipper + pdf-plus) | Readwise Reader | Raindrop |
|---|---|---|---|
| Cost | Free, MIT / open source [[obsidian-web-clipper]] | ~$9.99/mo (Full plan) [[readwise-reader-review-2026-worth-it-speed-reading-lounge]] | Freemium |
| Durability / lock-in | Best: Markdown in your vault, survives plugin death [[obsidian-pdf-plusreadmemd-at-main-ryotaushioobsidian-pdf-plus-github]] | Append-only Jinja2 export to vault; removable seam [[how-does-the-readwise-to-obsidian-export-integration-work-readwise-docs]] | Poor: no Markdown export path; plugin abandoned [[read-it-later-alternatives-after-omnivore-shutting-down-daniel-prindii]] |
| Mobile capture UX | Weak: Android blocks extensions; iOS Safari-only [[official-web-clipper-on-android-help-obsidian-forum]] | Best: 4.6★ iOS app, share-sheet save in <2s [[readwise-reader-review-2026-worth-it-speed-reading-lounge]] | Good: polished mobile reader [[pocket-shuts-down-in-july-2025-the-10-best-alternatives]] |
| Content breadth | Web + PDF (separate flows) | Unified: web, PDF, EPUB, RSS, newsletters, YouTube transcripts [[readwise-reader-the-first-read-it-later-app-built-for-power-readers]] | Bookmarks + reader + highlights [[pocket-shuts-down-in-july-2025-the-10-best-alternatives]] |
| Frontmatter control | One-to-one with your vault schema [[the-obsidian-web-clipper-fixes-everything-obsidian-rocks]] | Templated Jinja2 → your YAML [[how-does-the-readwise-to-obsidian-export-integration-work-readwise-docs]] | None native |
| Vendor risk | None (it's your files) | Lower than peers: $17M seed May 2025, best-capitalized [[readwise-funding-complete-analysis-extruct-ai]] | Closed-source SaaS |

The verdict is conditional, and the deciding fact is durability. Two read-it-later services died inside fourteen months — Omnivore (Oct 2024) and Pocket (Jul 2025), both *not* venture-backed — while Markdown in a local vault is immortal [[read-it-later-alternatives-after-omnivore-shutting-down-daniel-prindii]] [[pocket-shuts-down-in-july-2025-the-10-best-alternatives]]. But the mortality argument should be *softened*: Readwise raised a $17M seed in May 2025 and is now the best-capitalized service in the category [[readwise-funding-complete-analysis-extruct-ai]]. So the right framing is not "they all die soon" but "Markdown in your vault is the only guaranteed-durable substrate, so the seam to any external tool must be removable with zero schema lock-in."

The opposing camp's strongest reason deserves its due: for a high-volume, multi-format reader, Readwise's unified ingestion (web, PDF, EPUB, RSS, newsletters, YouTube transcripts) and <2s mobile share-sheet raise the *capture rate*, and capturing more of what you actually read can net more learning even at some durability cost. So durability bounds *how* you integrate it, not categorically *whether* a heavy mobile reader should.

Given that: **native-first if you are desktop-primary** (durability, zero cost, zero lock-in); **Readwise as a removable capture front-end if you are mobile-heavy**, provided its Jinja2 export writes your native frontmatter (`Type: Source`, `reading-status: inbox`, `Parent → 00000`) so removing it later leaves the vault uncorrupted [[how-does-the-readwise-to-obsidian-export-integration-work-readwise-docs]]. Raindrop is at best a manual triage buffer; its Obsidian path is HTML/CSV with no Markdown export, so it cannot be your backbone [[read-it-later-alternatives-after-omnivore-shutting-down-daniel-prindii]].

If you do adopt Readwise, the seam is only genuinely removable if its export writes your frontmatter, not Readwise's defaults. Set the plugin's export template so the YAML header maps to your schema [[how-does-the-readwise-to-obsidian-export-integration-work-readwise-docs]]:

```jinja2
---
Type: Source
Parent: "[[Resources/Reading/00000]]"
reading-status: inbox
source: {{ source_url }}
author: {{ author }}
ideas-extracted: []
tags: [reading-inbox, readwise-import]
---
{% for highlight in highlights %}
> {{ highlight.text }}
{% endfor %}
```

The `reading-inbox, readwise-import` tags are the removal seam: delete the plugin later and you can find every Readwise-origin note in one query, while the notes themselves stay valid `Type: Source` entries with `reading-status` intact — zero lock-in at the note-schema level.

Of the named alternatives, Omnivore is out (shut down Oct 2024) and Matter, though a viable mobile-first reader with Obsidian highlight export, sits in the same external-seam/lock-in category as Readwise without Readwise's funding durability [[pocket-shuts-down-in-july-2025-the-10-best-alternatives]] — so it is not separately tabled; if you want an external reader, the decision is Readwise vs. native, and Matter does not change it.

### E. Plugin ledger — all already installed

| Plugin | Role in intake | New config needed |
|---|---|---|
| Obsidian Web Clipper | Capture web articles → `Type: Source` Markdown | Template → `Inbox/`, stamp `reading-status: inbox` |
| pdf-plus | Annotate PDFs as Markdown backlinks | "Quote in callout" template; colour palette (v2) |
| QuickAdd | One-action "New Source" command | Capture/Template choice + Templater handoff |
| Templater | Stamp frontmatter, suggesters | The §8.C Source snippet; clone it for Idea notes against the §8.B schema |
| Dataview | Queue / count / progress dashboards | Enable inline + JS queries |
| Modal Forms | Optional structured capture form → frontmatter | Map fields to the §8.A schema (v2) |
| Kanban | Optional board view of `reading-status` | Columns = the lifecycle states (v2) |
| spaced-repetition | `{{curly}}` cloze on Idea notes (v2) | **Disable `==`/`**` as cloze markers** |
| periodic-notes | Weekly review = look at the dashboards | Embed dashboards in the weekly template |

The only genuinely new pieces are one enum value (`Source`), one frontmatter field (`reading-status`), one folder (`Inbox/`), one template, and one settings toggle. That is a small surface area for a complete pipeline.

### F. Phase 1 — the minimal build (an afternoon's work)

Build *only* this first. The goal is to defeat the collector's fallacy with the least friction and to prove the habit sticks before adding machinery.

1. **One folder:** `Inbox/` for captured Source notes.
2. **One new Type value:** `Source`, via the Templater snippet that sets `Type: Source` and `reading-status: inbox` and defers `Parent`/`Area` to the digest step.
3. **One capture tool:** Obsidian Web Clipper, template-mapped to the above frontmatter. (PDFs: drop into `Inbox/`, create a companion Source note.)
4. **One digest habit:** when you read a Source, highlight with `==`, then for the keepers write *one* own-words `Type: Idea` note that links back, and set the Source's `reading-status` to `integrated`.
5. **Two dashboards:** the reading-queue table and the inbox count from §7.A.

That is the entire v1: one new Type, one new field, one folder, one tool, one habit. It can be stood up in under an hour, and it directly answers your request — capture to a read-it-later state (`inbox`), digest later (read + write an Idea), highlight what matters (`==` / pdf-plus), learn (the Idea note in your own words, linked).

### G. Deferred features — each behind a named felt-pain trigger

This table is the discipline made concrete: every row is a real, defensible feature, and every one is *off* until a specific pain forces it on.

| Addition | Add only when | Why deferred |
|---|---|---|
| pdf-plus "Quote in callout" + Better Search Views annotation workflow | You read enough papers that companion-note highlighting feels clumsy | Real value, but only at paper volume |
| The `processed`/`integrated`/`dropped` states + `ideas-extracted` field | "Read vs not read" stops showing you *where* ideas stall | `integrated` duplicates "an Idea note exists," already queryable; a status that drifts lies to the dashboard |
| WIP-cap + staleness-alarm dashboards | A plain inbox *count* has demonstrably failed to move you for 3+ months | Numbers (10/14) are uncalibrated; alarms become noise; dashboards are a procrastination surface |
| Spaced repetition (`{{curly}}` cloze, Idea notes, inference-level prompts) | Specific ideas keep slipping and you want them top-of-mind | High recurring authoring cost; restrict to a curated few, never the default flow |
| Optional `maturity` field on Idea notes (`seedling`/`budding`/`evergreen`) | The Idea folder is large enough that "which ideas are still thin?" is a real question [[growing-the-evergreens]] | Pure overhead until the graph is big; the value is a query that surfaces under-developed ideas to grow |
| Maps of Content (MOCs — hub notes linking related Ideas) for a topic | You hit Nick Milo's "mental squeeze point" — too many related Idea notes to juggle [[maps-of-content-effortless-organization-for-notes-obsidian-rocks]] | Emergent and disposable; create only when the pain appears |
| Readwise front-end for mobile capture | Your reading is genuinely phone-heavy and the clipper's mobile friction bites | Adds a subscription and a seam to maintain; native is durable and free |
| Readwise Daily Review as a second loop | Never (run it *or* the vault's spaced repetition, not both) | Two queues split the loop and review un-processed highlights |
| Full Zettelkasten (Folgezettel IDs, ZK folder, atomicity gate) | You become a high-volume publisher | By Doto's own test, unnecessary for learning/ideas; atomicity-as-gate causes paralysis |

### H. What to skip outright

- **A separate fleeting-note tier** — it is functionally a `Source` with `reading-status: reading`. (One distinction is worth naming: the design's Inbox is a *reading* inbox of captured sources; your own half-formed fragments — the seedling ideas that precede a full Idea note — belong in a separate *writing* inbox, best handled as a low-`maturity` Idea note drained during your periodic-notes review rather than a new Type [[a-writing-inbox-for-transient-and-incomplete-notes]].)
- **Sub-folders and tags on the inbox** — the flat lane is the point [[i-solved-my-read-it-later-bookmarks-problem-with-a-reading-inbox-in-obsidian]].
- **Clipping everything** — "when in doubt, don't clip; add a link instead." Capture the value in a note and move on [[the-obsidian-web-clipper-fixes-everything-obsidian-rocks]].
- **AI auto-summarization at capture** (the clipper's Interpreter, Readwise's Ghostreader) as your *digest* — a machine summary is highlighting with extra steps (see §9.D). Use it, if at all, only as a pre-read filter, never as the thing that moves a Source to `integrated`.

## 9. Opinionated Synthesis

### A. The committed recommendation

Build the generative-act pipeline, and build the smallest version of it first. Capture cheaply into a flat `Inbox/` — Web Clipper for web, pdf-plus for papers — as `Type: Source` notes carrying a `reading-status` lifecycle, and treat that queue as a staging area whose only purpose is to be drained. The unit of "done" is an extracted `Type: Idea` note, in your own words, that backlinks to its Source; highlighting is a pointer to where that work is owed. Link your Idea notes across Areas — the empirically-backed idea engine — and use the daily count of new Idea notes as your best single progress metric. Reinforce a curated subset, if you choose, with `{{curly}}` cloze cards using elaborative prompts. Keep everything native Markdown, because the substrate is the one thing guaranteed to outlive every app.

This is not a hedge between Forte and Dunlosky, between Ahrens and zettelkasten.de, between native and external. It is a decision: when the evidence is lopsided, you follow it and demote the popular-but-weak practice to a supporting role.

### B. The expert disagreements, settled

- **Highlight (Forte) vs. retrieve (Dunlosky):** Both right about different objects. Highlighting is a cheap filter (d=0.44, weakest but above baseline), a useless finish. Lead with the cognitive science; absorb Progressive Summarization as the on-ramp [[frontiers-a-meta-analysis-of-ten-learning-techniques]] [[strengthening-the-student-toolbox]].
- **Three note tiers vs. collapse them:** Two tiers — new `Type: Source`, existing `Type: Idea`. A third "literature note" tier is just a Source still `reading`.
- **Do you need a Zettelkasten?** No slip-box ceremony. Graft three light principles (concept-oriented title, atomicity-as-outcome, ≥1 cross-domain link) onto the existing Idea Type. The payoff is real, now evidence-backed (g=0.72), and accrues without Folgezettel IDs [[studying-and-constructing-concept-maps-a-meta-analysis-educational-psychology-re]].
- **Native vs. external:** Native-first on durability and zero lock-in; Readwise is a defensible mobile capture front-end whose output must be native frontmatter and whose Daily Review you must *not* run as a second brain [[readwise-funding-complete-analysis-extruct-ai]].
- **Spaced repetition worth it?** Yes, scoped: per-idea, elective, elaborative prompts only (factual = near-zero transfer), never over raw sources [[pan-s-c-rickard-t-c-2018-transfer-of-test-enhanced-learning-meta-analytic-review]].

### C. Build v1, not v2 — and the one genuine unknown

The biggest risk to this project (Section 1.D) is building an elaborate system that becomes another dashboard you admire and never use. So the minimal v1 — one folder, one `Type: Source`, one `reading-status`, the Web Clipper, two dashboards, and the one-Idea-per-source habit — is not a stepping stone you tolerate on the way to the "real" system. For most people it *is* the real system. Add the richer states, the alarms, spaced repetition, and Maps of Content only when v1's daily use surfaces the specific pain each solves. Grow the machinery to fit the habit; never grow the habit to fit the machinery.

The core tradeoff, stated honestly: a minimal system will let some captured items rot unread. The richer design is correct that this happens — but the cure it reaches for (more states, caps, alarms) treats the rot as an engineering problem, when it is mostly a *demand* problem. You captured more than you will ever process, and no lifecycle changes that. An unread item that ages out is the system telling you the truth — you did not actually want to read it; deleting it guilt-free is the right response, not building an alarm to nag you into reading something you were never going to value.

The one parameter the evidence cannot settle for you is the calibration of any future WIP cap and staleness threshold to *your* real intake rate. If you ever build the v2 alarms, start at roughly ten items and fourteen days, watch the digestion-progress query for a month, and tune. Everything else here is settled by the evidence; that one number is settled only by your own behavior.

### D. Forward-looking analysis

Read the pipeline you build today not as a reading tracker but as a *semantic-network cultivation* engine. In year one it will feel like a queue with a nagging count. But the return compounds nonlinearly: as your Idea graph grows denser and more cross-linked, you are building the kind of connected, low-modularity semantic network that predicts creative output [[memory-and-creativity-a-meta-analytic-examination-of-the-relationship-between-me]], and constructing those links is itself a high-yield learning move [[studying-and-constructing-concept-maps-a-meta-analysis-educational-psychology-re]]. Matuschak's observation that unexpected connections "tend to appear where they're not quite so expected" is not mysticism — it is spreading activation across a graph you spent a year wiring. Over time, the "most-linked Idea notes" query becomes a map of your own intellectual centre of gravity.

Two near-term shifts point the same way. First, the read-it-later category is consolidating around its users — Omnivore and Pocket are gone, Readwise has taken its first institutional money, and Obsidian itself now ships Bases as a native Dataview alternative [[read-it-later-alternatives-after-omnivore-shutting-down-daniel-prindii]] [[readwise-funding-complete-analysis-extruct-ai]]. A design anchored in vault-native Markdown with a removable external seam is the only one robust to that churn: when the next tool dies or is born, you swap a capture front-end and a Jinja2 template, and your Sources, Ideas, links, highlights, and review schedules all stay put. You are betting on the file format, not the feature set — the only thing in this stack guaranteed to still be readable in a decade.

Second, AI: clippers and readers increasingly auto-summarize on capture, and that is a trap for this design [[write-about-what-you-read-to-internalize-texts-deeply]]. The line to hold is not "no AI" but the boundary between substitution and scaffolding. AI that performs the generative act *for* you — writes the summary, writes the Idea note — steals the learning, while AI that *scaffolds* your generation — drafting retrieval prompts you then answer in your own words, surfacing candidate connections or contradictions you then resolve — can lower the activation energy that stalls the pipeline. So use AI to surface candidate connections between your existing Idea notes, or to generate inference-level questions you answer yourself, and never to write the Idea note for you.

The deepest forward-looking claim is the simplest. Your original question framed highlighting as the act where learning happens. The evidence says learning happens one step *past* the highlight — in the moment you close the source and write, in your own words, the thing it made you think. As machines get cheaper at acquiring and compressing information, that one un-automatable act — turning a source you read into an idea you own — becomes the scarce and valuable one. Build a system that makes that step the cheapest, most visible, and most rewarded thing in your workflow, and let the rest of your instinct — capture, queue, highlight — be exactly the friction-free scaffolding it was always meant to be.
