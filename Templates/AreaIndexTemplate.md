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
let title = tp.file.title;
if (title.startsWith("Untitled")) {
  title = (await tp.system.prompt("Area name")) ?? title;
}
await tp.file.rename(title);
const areaTag = "area/" + slugify(title);
-%>
---
Parent: "[[Index|Link]]"
Type: Index
tags:
  - <% areaTag %>
---
# <% title %>

## Projects
```dataview
TABLE Project AS "Project", join(filter(file.etags, (t) => startswith(t, "#topic/")), ", ") AS "Topics"
FROM "PROJECTS"
WHERE file.name = "00000" AND contains(file.tags, "#<% areaTag %>")
```

## Ideas
```dataview
TABLE status AS "Status"
WHERE Type = "Idea" AND contains(file.tags, "#<% areaTag %>")
SORT status ASC
```

## Resources
```dataview
LIST
FROM "RESOURCES"
WHERE contains(file.tags, "#<% areaTag %>")
```

## Sources
```dataview
LIST
WHERE Type = "Source" AND contains(file.tags, "#<% areaTag %>")
```
