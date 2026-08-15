---
Parent: "[[Index|Link]]"
Type: Documentation
tags:
  - area/meta
---
# Capture — Setup and Usage

How to get anything — article, paper, link someone mentions — into `Inbox/` as a properly stamped `Type: Source` note, from any device. Four routes, all landing in the same reading queue on [[Index]].

*Capturing your own thoughts rather than someone else's material is a different path — see [§5](#5-capturing-your-own-ideas) at the bottom.*

| Route | When | Setup status |
|---|---|---|
| QuickAdd hotkey | You're inside Obsidian (desktop or mobile) | ✅ configured |
| iOS share sheet | You're reading on an iPhone/iPad | ✅ configured (test once) |
| **Android share sheet → MacroDroid** | You're reading on Android | ⚠️ needs one-time setup below |
| Web Clipper | You're reading in a desktop browser | ⚠️ needs one-time setup below |

The contract every capture must satisfy: frontmatter with `Type: Source`, `reading_status: inbox`, `Parent: "[[Inbox/00000|Link]]"`, `source:` = the URL. If any of these is missing or misspelled, the note silently skips every dashboard.

---

## 1. QuickAdd hotkey (already configured)

Press `Cmd/Ctrl+Shift+S` anywhere in Obsidian → prompted for title, then URL → stamped Source note appears in `Inbox/`. On mobile, add the same command to the toolbar: Settings → Toolbar → add "QuickAdd: New Source".

## 2. iOS share sheet (already configured)

Share any page → choose Obsidian → the note lands in `Inbox/` (default new-note folder) and the Templater folder template stamps it: URL detected from the shared text, title taken from the filename.

**Test it once:** share an article, let it sync, and check the note has the full frontmatter. If it arrives raw (Templater sometimes skips files created with content already in them), open it and insert `Templates/SourceTemplate` manually — the same detection logic runs and stamps it.

## 3. Android share sheet → MacroDroid — one-time setup

**Why this is different from iOS.** Obsidian's Android share target is *append-only by design*. Share a page to Obsidian on Android and you get "Add text to file: Index / Choose a file… / Today's daily note" — it can only append to a note that already exists. There is no create-a-new-note path, and there hasn't been since the feature was first requested in 2022. So on Android the share sheet cannot produce a Source note on its own.

The fix is to share to **MacroDroid** instead of to Obsidian. MacroDroid writes the finished `.md` file straight into `Inbox/` on the phone's filesystem — Obsidian never has to open, and the note arrives already stamped, so it needs no Templater pass at all.

### Prerequisites

1. Install **MacroDroid** (Play Store). The free tier caps you at 5 macros; this uses 1.
2. Grant it all-files access: Android Settings → Apps → MacroDroid → Permissions → Files and media → **Allow management of all files**. Without this the write silently fails.
3. Exempt it from battery optimisation (MacroDroid will prompt), or Android may kill it before the write lands.
4. Know where the vault folder lives on the phone — you don't have to type the path, the Write-to-File action has a folder browser.

### Build the macro

Name it **Capture to PARA Inbox**. One trigger, three actions, no constraints. Magic text is written in curly braces — `{shared_text}`, `{lv=title}` — and it's safest to insert it from the magic-text picker rather than typing it.

**Trigger** — Applications → *Text Shared to MacroDroid*, with Text to Match set to `*` so every share matches.

**Action 1 — Text Manipulation** (pull the URL out, since some apps share `Title\nURL` rather than a bare link)

| Field | Value |
|---|---|
| Operation | Replace All, with **regex enabled** |
| Input | `{shared_text}` |
| Pattern | `(?s)^.*?(https?://\S+).*$` |
| Replace with | `$1` |
| Output variable | `url` |

**Action 2 — Text Manipulation** (turn the page title into a safe filename)

| Field | Value |
|---|---|
| Operation | Replace All, regex enabled |
| Input | `{shared_text_subject}` |
| Pattern | `[\\/:*?"<>\|#^\[\]]` |
| Replace with | *(leave empty)* |
| Output variable | `title` |

`{shared_text_subject}` is the page title Android sends alongside the URL — no dictionary or intent-extras digging needed. The character class strips everything Android and Obsidian refuse in a filename: `\ / : * ? " < > | # ^ [ ]`.

**Action 3 — Write to File**

- Directory: browse to the vault → `Inbox`
- Filename: `{lv=title}.md`
- **Append: OFF** (create/overwrite)
- Content:

```
---
Type: Source
Parent: "[[Inbox/00000|Link]]"
reading_status: inbox
source: {lv=url}
author: ""
ideas-extracted: []
tags: []
---
# {lv=title}

{lv=url}

## Highlights
<!-- ==highlights== (web) or pdf-plus backlinks (papers) land here while reading -->

## Digest — owes an Idea note
- [ ] Cluster highlights by concept (not by the source's chapter order)
- [ ] Write Idea note(s) in your own words; pick this Source in the Idea template's "Source note(s) this idea came from" prompt — `ideas-extracted` here and `source-notes` there are then filled in automatically, and the template offers to set `reading_status: integrated`
```

That's the whole macro. Share → note on disk, zero taps beyond picking MacroDroid.

### Two optional actions worth adding

Neither is required, but both close a real gap:

- **Empty-title guard.** Insert between Actions 2 and 3: *If `title` is empty → Set Variable* `title` = `Capture {year}-{month_digit}-{dayofmonth} {hour}{minute}`. Some apps share a bare URL with no subject; without this you get a file literally named `.md`, which Obsidian hides. Pick the date tokens from the magic-text picker so you get whatever your version calls them.
- **Toast** at the end: `Captured → Inbox/{lv=title}.md`. Confirms the write landed, and shows you immediately when the title came through empty or mangled.

### Test it

Share a page from Chrome → pick MacroDroid. Open Obsidian on the phone and check [[Index]] → Reading queue. The note should be there with `reading_status: inbox` and an Age of 0.

### Things that will bite you

- **Sync timing.** MacroDroid writes to disk instantly, but *when that reaches your other devices depends on your sync*. Syncthing picks it up immediately with Obsidian closed. Relay and Obsidian Sync are plugins running inside Obsidian, so nothing propagates until you open Obsidian on the phone at least once. The note is never lost either way — it's sitting in `Inbox/` on the phone.
- **Relay shares folders, not vaults.** If you're on Relay, `Inbox/` has to be one of the shared folders or captures stay phone-only.
- **Duplicate titles overwrite.** Append is off, so sharing the same article twice with the same title replaces the first note. Add a date to the title if that matters.
- **A share with no subject gives you `.md`.** `{shared_text_subject}` is empty when the sharing app sends only a URL — the empty-title guard above is the fix.
- **No double-stamping.** `Templates/SourceTemplate` checks for existing `Type: Source` frontmatter and hands the file back untouched, so a MacroDroid note is safe if it ever passes through Templater.
- **The obsidian-git toast** ("Can't find a valid git repository") is unrelated to capture — the Git plugin has no repo on Android. Disable it under mobile-specific plugin settings.

## 4. Web Clipper — one-time setup

The official **Obsidian Web Clipper** extension. Unlike the share route, it can capture the article *content* and your highlights, not just the URL.

### Install
- **Desktop:** install "Obsidian Web Clipper" from your browser's extension store (Chrome/Firefox/Edge).
- **iPhone/iPad:** install the Obsidian Web Clipper app from the App Store, then enable it: Settings → Apps → Safari → Extensions. It then appears in Safari's share sheet.
- **Android:** only Firefox for Android supports extensions — the clipper is on addons.mozilla.org and works there. Chrome on Android can't run it; use the MacroDroid route above.

In the extension's settings, connect it to the `SecondBrainPARA` vault. `paravault-clipper-config.json` in the vault root is an importable copy of the template below.

### Create the "Source" template
In the clipper's settings, create a new template (call it `Source`) and set:

**Behavior:** Create new note. **Note location:** `Inbox`. **Note name:** `{{title}}`.

**Properties** (add each one; types: text unless noted):

| Property          | Value                   | Property type |
| ----------------- | ----------------------- | ------------- |
| `Type`            | `Source`                | text          |
| `Parent`          | `[[Inbox/00000\|Link]]` | text          |
| `reading_status`  | `inbox`                 | text          |
| `source`          | `{{url}}`               | text          |
| `author`          | `{{author}}`            | text          |
| `ideas-extracted` | *(leave empty)*         | list          |
| `tags`            | *(leave empty)*         | list          |

Spelling matters: `reading_status` with an underscore, `Source` capitalised. A typo here and the capture never shows up in the queue.

**Note content:**

```
# {{title}}

## Highlights
{{highlights}}

## Digest — owes an Idea note
- [ ] Cluster highlights by concept (not by the source's chapter order)
- [ ] Write Idea note(s) in your own words; backlink this Source
- [ ] Add the Idea link to `ideas-extracted`, set `reading_status: integrated`
```

(If you'd rather keep the full article text for offline reading, add `{{content}}` under the title — but the queue works fine without it, and lighter notes sync faster.)

Set `Source` as the default template so one click clips correctly.

### Use it
- **Desktop:** click the clipper icon (or `Cmd/Ctrl+Shift+O` if you enable its shortcut) → check the template says `Source` → Add to Obsidian. Select text on the page first and use the highlighter mode if you want highlights captured into the `## Highlights` section.
- **iOS:** Share → Obsidian Web Clipper → clip. Same template applies.

The Templater guard in `SourceTemplate` recognises clipper notes as already stamped and leaves them alone — no double-processing.

---

## 5. Capturing your own ideas

Everything above is for material someone else made. An idea of your own never touches `Inbox/` — it has no `source:`, so it isn't a Source, and the reading queue would be the wrong place for it.

**Press `Cmd/Ctrl+Shift+I`** — QuickAdd "New Idea" asks two things, the idea's title and its area, and drops a note into `IDEAS/` stamped `Type: Idea`, `status: seed`, `source-notes: []`. That's the whole capture. Topics and maturity are deliberately deferred: the Index's "Seeds needing development" table will keep showing the note until you go back and develop it, which is the nag doing its job.

On mobile, add the command to the toolbar the same way as New Source: Settings → Toolbar → add "QuickAdd: New Idea".

**When to use the full `IdeaTemplate` instead** (insert it via Templater, or run it on an existing note): when you're sitting down to write properly, and especially when the idea came *out of* something you read. Its source picker links the Source note both ways — `source-notes` on the idea, `ideas-extracted` on the source — and offers to flip `reading_status: integrated` in the same pass. Both kinds of idea end up in `IDEAS/`; where an idea came from is recorded in its frontmatter, not in its location.

---

## After capture (all routes)

Captured ≠ done. The note sits in the reading queue on [[Index]] with its age counting up. The work it owes: read it, then write at least one `Type: Idea` note **in your own words**, backlink both ways, and flip `reading_status: integrated`. The queue warns above 10 undigested sources; the stale table names anything older than 14 days. If it's been stale twice, delete it — that's a decision too.
