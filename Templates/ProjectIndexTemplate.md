<%*
function slugify(s) {
  return String(s).trim().toLowerCase().replace(/[^a-z0-9/]+/g, "-").replace(/^-+|-+$/g, "");
}
function yamlList(items) {
  return items.length ? items.map(t => "  - " + t).join("\n") : "  []";
}
function allFrontmatterTags(prefix) {
  const out = new Set();
  for (const f of app.vault.getMarkdownFiles()) {
    const tags = app.metadataCache.getFileCache(f)?.frontmatter?.tags;
    if (Array.isArray(tags)) {
      for (const t of tags) if (String(t).startsWith(prefix)) out.add(String(t));
    }
  }
  return Array.from(out).sort();
}
async function pickAreaHub() {
  const hubs = app.vault.getMarkdownFiles()
    .filter(f => f.path.startsWith("AREAS/") && !f.path.slice("AREAS/".length).includes("/"));
  const labels = hubs.map(h => h.basename).concat(["Other (new area)..."]);
  const values = [...hubs, null];
  const pick = await tp.system.suggester(labels, values, false, "Area (exactly one)");
  if (pick === null || pick === undefined) {
    const name = await tp.system.prompt("New area name (create its hub note in AREAS/ too):");
    return { hubName: name ?? "Unsorted", tag: "area/" + slugify(name ?? "unsorted") };
  }
  const fm = app.metadataCache.getFileCache(pick)?.frontmatter;
  const tag = (fm?.tags ?? []).map(String).find(t => t.startsWith("area/"))
    ?? ("area/" + slugify(pick.basename));
  return { hubName: pick.basename, tag: tag };
}
async function pickTopicTags() {
  const chosen = [];
  for (;;) {
    const existing = allFrontmatterTags("topic/").filter(t => !chosen.includes(t));
    const labels = [...existing, "+ New topic...", "Done (" + chosen.length + " selected)"];
    const values = [...existing, "__new__", "__done__"];
    const pick = await tp.system.suggester(labels, values, false, "Topic tags (optional)");
    if (pick === "__done__" || pick === null || pick === undefined) break;
    if (pick === "__new__") {
      const raw = await tp.system.prompt("New topic (nest with '/', e.g. programming/python):");
      if (raw) chosen.push("topic/" + slugify(raw));
    } else {
      chosen.push(pick);
    }
  }
  return chosen;
}
/* The project's kanban board, created alongside this index.

   This deliberately does NOT invoke `TasksTemplate`: that template reads the
   project index's frontmatter out of the metadata cache to inherit the area
   tag, and at this moment the index note has not been written yet — the cache
   would come back empty and the board would land with `tags: []`. Same class of
   race as the v2.0 frontmatter bug (SYSTEM.md §3). So the board is emitted from
   the variables already in hand.

   ⚠️ KEEP IN SYNC with `TasksTemplate.md` — the two produce the same skeleton
   and `TasksTemplate` is still the manual path for a project that has no board. */
async function createBoard(folderPath, folderName, areaTag) {
  const existing = app.vault.getMarkdownFiles()
    .filter(f => f.parent?.path === folderPath)
    .find(f => app.metadataCache.getFileCache(f)?.frontmatter?.Type === "Tasks");
  if (existing) return;
  const path = `${folderPath}/${folderName} Tasks.md`;
  if (app.vault.getAbstractFileByPath(path)) return;
  const body = [
    "---",
    `Parent: "[[${folderPath}/00000|Link]]"`,
    "Type: Tasks",
    `Project: "${folderName}"`,
    "kanban-plugin: board",
    "tags:",
    areaTag ? "  - " + areaTag : "  []",
    "---",
    "",
    "## Backlog",
    "",
    "## In Progress",
    "",
    "## Done",
    "",
    "%% kanban:settings",
    "```",
    '{"kanban-plugin":"board","list-collapse":[false,false,false],"show-checkboxes":false}',
    "```",
    "%%",
    "",
  ].join("\n");
  try {
    await app.vault.create(path, body);
    new Notice(`Board created: ${folderName} Tasks`);
  } catch (e) {
    new Notice(`Board not created: ${e.message}`);
  }
}
let title = tp.file.title;
if (title.startsWith("Untitled")) { title = "00000"; }
await tp.file.rename(title);

const area = await pickAreaHub();
const topics = await pickTopicTags();
const tagBlock = yamlList([area.tag, ...topics]);

await createBoard(tp.file.folder(true), tp.file.folder(), area.tag);
-%>
---
Parent: "[[AREAS/<% area.hubName %>|Link]]"
Type: Index
Project: "<% tp.file.folder() %>"
tags:
<% tagBlock %>
---
# <% tp.file.folder() %>

## Tasks board
```dataview
LIST
WHERE file.folder = this.file.folder AND Type = "Tasks"
```

## Project files by topic
```dataviewjs
const folder = dv.current().file.folder;
const pages = dv.pages(`"${folder}"`).where(p => p.file.name !== "00000" && p.Type !== "Tasks");
const groups = {};
for (const p of pages) {
  const topics = Array.from(p.file.etags ?? []).map(String).filter(t => t.startsWith("#topic/"));
  if (topics.length === 0) { (groups["(no topic)"] ??= []).push(p); }
  for (const t of topics) { (groups[t] ??= []).push(p); }
}
const keys = Object.keys(groups).sort();
if (keys.length === 0) { dv.paragraph("*No project files yet.*"); }
for (const key of keys) {
  dv.header(3, String(key).replace("#topic/", ""));
  dv.table(["Note", "Type"], groups[key].map(p => [p.file.link, p.Type]));
}
```

## All project files
```dataview
TABLE Type
WHERE file.folder = this.file.folder AND file.name != "00000"
```
