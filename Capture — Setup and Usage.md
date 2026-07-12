---
Parent: "[[Index|Link]]"
Type: Documentation
tags:
  - area/meta
---
# Capture — Setup and Usage

How to get anything — article, paper, link someone mentions — into `Inbox/` as a properly stamped `Type: Source` note, from any device. Three routes, all landing in the same reading queue on [[Index]].

| Route | When | Setup status |
|---|---|---|
| QuickAdd hotkey | You're inside Obsidian (desktop or mobile) | ✅ configured |
| Phone share sheet | You're reading on your phone | ✅ configured (test once) |
| Web Clipper | You're reading in a browser | ⚠️ needs one-time setup below |

The contract every capture must satisfy: frontmatter with `Type: Source`, `reading_status: inbox`, `Parent: "[[Inbox/00000|Link]]"`, `source:` = the URL. If any of these is missing or misspelled, the note silently skips every dashboard.

---

## 1. QuickAdd hotkey (already configured)

Press `Cmd/Ctrl+Shift+S` anywhere in Obsidian → prompted for title, then URL → stamped Source note appears in `Inbox/`. On mobile, add the same command to the toolbar: Settings → Toolbar → add "QuickAdd: New Source".

## 2. Phone share sheet (already configured)

Share any page → choose Obsidian → the note lands in `Inbox/` (default new-note folder) and the Templater folder template stamps it automatically: URL detected from the shared text, title taken from the filename.

**Test it once:** share an article from your phone, let it sync, and check the note has the full frontmatter. If it arrives raw (Templater sometimes skips files created with content already in them), open it and insert `Templates/SourceTemplate` manually — the same detection logic runs and stamps it.

## 3. Web Clipper — one-time setup

The official **Obsidian Web Clipper** extension. Unlike the share route, it can capture the article *content* and your highlights, not just the URL.

### Install
- **Desktop:** install "Obsidian Web Clipper" from your browser's extension store (Chrome/Firefox/Edge).
- **iPhone/iPad:** install the Obsidian Web Clipper app from the App Store, then enable it: Settings → Apps → Safari → Extensions. It then appears in Safari's share sheet.
- **Android:** extensions require Firefox for Android (Chrome on Android doesn't support them). Otherwise use the share-sheet route.

In the extension's settings, connect it to the `SecondBrainPARA` vault.

### Create the "Source" template
In the clipper's settings, create a new template (call it `Source`) and set:

**Behavior:** Create new note. **Note location:** `Inbox`. **Note name:** `{{title}}`.

**Properties** (add each one; types: text unless noted):

| Property | Value | Property type |
|---|---|---|
| `Type` | `Source` | text |
| `Parent` | `[[Inbox/00000\|Link]]` | text |
| `reading_status` | `inbox` | text |
| `source` | `{{url}}` | text |
| `author` | `{{author}}` | text |
| `ideas-extracted` | *(leave empty)* | list |
| `tags` | *(leave empty)* | list |

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

## After capture (all routes)

Captured ≠ done. The note sits in the reading queue on [[Index]] with its age counting up. The work it owes: read it, then write at least one `Type: Idea` note **in your own words**, backlink both ways, and flip `reading_status: integrated`. The queue warns above 10 undigested sources; the stale table names anything older than 14 days. If it's been stale twice, delete it — that's a decision too.
