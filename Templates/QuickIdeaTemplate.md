<%*
/* Fast capture for an idea of your own. Two prompts: title, then area.
   Topics, maturity and source links are deliberately deferred — the note
   lands as `status: seed` and the Index nags you until you develop it.
   Use `IdeaTemplate` instead when you're sitting down to write properly. */
function slugify(s) {
  return String(s).trim().toLowerCase().replace(/[^a-z0-9/]+/g, "-").replace(/^-+|-+$/g, "");
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
let title = tp.file.title;
if (title.startsWith("Untitled")) {
  title = (await tp.system.prompt("Idea — one line")) ?? title;
}
if (title !== tp.file.title) { await tp.file.rename(title); }

const area = await pickAreaHub();
-%>
---
Parent: "[[AREAS/<% area.hubName %>|Link]]"
Type: Idea
status: seed
source-notes: []
tags:
  - <% area.tag %>
---
# <% title %>


---
## Related notes (shared topics)
```dataviewjs
const here = dv.current();
const myTopics = Array.from(here.file.etags ?? []).map(String).filter(t => t.startsWith("#topic/"));
if (myTopics.length === 0) {
  dv.paragraph("*No topic tags on this note yet — add some when you develop it.*");
} else {
  const related = dv.pages().where(p =>
    p.file.path !== here.file.path &&
    Array.from(p.file.etags ?? []).map(String).some(t => myTopics.includes(t)));
  if (related.length === 0) {
    dv.paragraph("*Nothing shares a topic with this note yet.*");
  } else {
    dv.table(["Note", "Type"], related.map(p => [p.file.link, p.Type]));
  }
}
```
