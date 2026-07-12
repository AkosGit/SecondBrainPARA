<%*
let title = tp.file.title;
if (title.startsWith("Untitled")) {
  title = "00000";
}
await tp.file.rename(title);

// ---- helpers ----
function slugify(s) {
  return s.trim().toLowerCase().replace(/[^a-z0-9/]+/g, "-").replace(/^-+|-+$/g, "");
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
  // Area hubs are the flat .md notes sitting directly inside AREAS/
  const hubs = app.vault.getMarkdownFiles()
    .filter(f => f.path.startsWith("AREAS/") && !f.path.slice("AREAS/".length).includes("/"));
  const labels = hubs.map(h => h.basename);
  labels.push("Other (new area)...");
  const values = [...hubs, null];
  const pick = await tp.system.suggester(labels, values, false, "Area (exactly one)");
  if (pick === null) {
    const name = await tp.system.prompt("New area name (remember to create its hub note in AREAS/):");
    return { hubName: name, tag: "area/" + slugify(name) };
  }
  const fm = app.metadataCache.getFileCache(pick)?.frontmatter;
  const tag = (fm?.tags ?? []).map(String).find(t => t.startsWith("area/")) ?? ("area/" + slugify(pick.basename));
  return { hubName: pick.basename, tag: tag };
}

async function pickTopicTags() {
  const chosen = [];
  while (true) {
    const existing = allFrontmatterTags("topic/").filter(t => !chosen.includes(t));
    const labels = [...existing, "New topic...", "Done (" + chosen.length + " selected)"];
    const values = [...existing, "__new__", "__done__"];
    const pick = await tp.system.suggester(labels, values, false, "Add topic tags (optional)");
    if (pick === "__done__" || pick === null) break;
    if (pick === "__new__") {
      const raw = await tp.system.prompt("New topic (nest with '/', e.g. programming/python):");
      if (raw) chosen.push("topic/" + slugify(raw));
    } else {
      chosen.push(pick);
    }
  }
  return chosen;
}
// -----------------

const area = await pickAreaHub();
const topics = await pickTopicTags();

await app.fileManager.processFrontMatter(tp.config.target_file, fm => {
  fm["Parent"] = `[[AREAS/${area.hubName}|Link]]`;
  fm["Type"] = "Index";
  fm["Project"] = tp.file.folder();
  fm["tags"] = [area.tag, ...topics];
});
-%>
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
  const topics = Array.from(p.file.etags ?? []).filter(t => String(t).startsWith("#topic/"));
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
