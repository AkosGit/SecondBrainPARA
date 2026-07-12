<%*
// Share-aware Source capture. Reached three ways:
//   1. QuickAdd "New Source" (Cmd/Ctrl+Shift+S) — empty file, prompts for everything
//   2. Android share -> Obsidian — URL already in the body, title in the filename
//   3. Templater folder-template trigger on any new file in Inbox/
const raw = await app.vault.read(tp.config.target_file);

// Guard: already a stamped Source (e.g. Web Clipper capture) — leave untouched.
if (/^---[\s\S]*?\bType:\s*Source\b[\s\S]*?---/.test(raw)) {
  return;
}

// URL: prefer one already present in the note (that's what a share deposits).
const urlMatch = raw.match(/https?:\/\/[^\s)\]>"']+/);
const url = urlMatch ? urlMatch[0] : ((await tp.system.prompt("URL or PDF path", "")) ?? "");

// Title: the filename if meaningful (a share names the note after the page),
// otherwise prompt and rename.
let title = tp.file.title;
if (title.startsWith("Untitled")) {
  title = (await tp.system.prompt("Source title")) ?? title;
  await tp.file.rename(title);
}

// Frontmatter via API so it is written correctly regardless of where
// Templater inserts the rendered body.
await app.fileManager.processFrontMatter(tp.config.target_file, fm => {
  fm["Type"] = "Source";
  fm["Parent"] = "[[Inbox/00000|Link]]";
  fm["reading_status"] = "inbox";
  fm["source"] = url;
  fm["author"] = "";
  fm["ideas-extracted"] = [];
  fm["tags"] = [];
});
-%>
# <% title %>

## Highlights
<%* /* ==highlights== (web) or pdf-plus backlinks (papers) land here while reading */ -%>

## Digest — owes an Idea note
- [ ] Cluster highlights by concept (not by the source's chapter order)
- [ ] Write Idea note(s) in your own words; backlink this Source
- [ ] Add the Idea link to `ideas-extracted`, set `reading_status: integrated`
