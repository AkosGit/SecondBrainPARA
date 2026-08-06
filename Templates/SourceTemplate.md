<%*
// Share-aware Source capture. Reached three ways:
//   1. QuickAdd "New Source" (Cmd/Ctrl+Shift+S) — empty file, prompts for everything
//   2. Phone share -> Obsidian — URL already in the body, title in the filename
//   3. Templater folder-template trigger on a new file in Inbox/
// The whole note is emitted through tR so the guard branch can hand back the
// original content untouched instead of letting Templater overwrite it.
const raw = await app.vault.read(tp.config.target_file);

if (/^---[\s\S]*?\bType:\s*Source\b[\s\S]*?^---/m.test(raw)) {
  // Already stamped (e.g. Web Clipper). Return the file exactly as it arrived.
  tR += raw;
} else {
  const urlMatch = raw.match(/https?:\/\/[^\s)\]>"']+/);
  const url = urlMatch ? urlMatch[0] : ((await tp.system.prompt("URL or PDF path", "")) ?? "");

  let title = tp.file.title;
  if (title.startsWith("Untitled")) {
    title = (await tp.system.prompt("Source title")) ?? title;
    await tp.file.rename(title);
  }

  tR += `---
Type: Source
Parent: "[[Inbox/00000|Link]]"
reading_status: inbox
source: ${url}
author: ""
ideas-extracted: []
tags: []
---
# ${title}

${url ? url + "\n" : ""}
## Highlights
<!-- ==highlights== (web) or pdf-plus backlinks (papers) land here while reading -->

## Digest — owes an Idea note
- [ ] Cluster highlights by concept (not by the source's chapter order)
- [ ] Write Idea note(s) in your own words; pick this Source in the Idea template's
      "Source note(s) this idea came from" prompt — \`ideas-extracted\` here and
      \`source-notes\` there are then filled in automatically, and the template offers
      to set \`reading_status: integrated\`
`;
}
-%>
