<%*
function slugify(s) {
  return String(s).trim().toLowerCase().replace(/[^a-z0-9/]+/g, "-").replace(/^-+|-+$/g, "");
}
function yamlList(items) {
  return items.length ? items.map(t => "  - " + t).join("\n") : "  []";
}
function yamlField(name, items) {
  return items.length ? name + ":\n" + items.map(t => "  - " + t).join("\n") : name + ": []";
}
function wikiLink(file, alias) {
  return `"[[${file.path.replace(/\.md$/, "")}|${alias ?? file.basename}]]"`;
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
/* Source notes this idea came from. Undigested sources are listed first. */
async function pickSourceNotes() {
  const chosen = [];
  for (;;) {
    const pool = app.vault.getMarkdownFiles()
      .filter(f => app.metadataCache.getFileCache(f)?.frontmatter?.Type === "Source")
      .filter(f => !chosen.some(c => c.path === f.path))
      .sort((a, b) => {
        const st = f => app.metadataCache.getFileCache(f)?.frontmatter?.reading_status === "integrated" ? 1 : 0;
        return (st(a) - st(b)) || (b.stat.ctime - a.stat.ctime);
      });
    if (pool.length === 0) break;
    const labels = pool.map(f => {
      const st = app.metadataCache.getFileCache(f)?.frontmatter?.reading_status;
      return f.basename + (st ? "  ·  " + st : "");
    }).concat(["Done (" + chosen.length + " selected)"]);
    const values = [...pool, "__done__"];
    const pick = await tp.system.suggester(
      labels, values, false, "Source note(s) this idea came from — Esc for none");
    if (pick === "__done__" || pick === null || pick === undefined) break;
    chosen.push(pick);
  }
  return chosen;
}
let title = tp.file.title;
if (title.startsWith("Untitled")) {
  title = (await tp.system.prompt("Title")) ?? title;
}
if (title !== tp.file.title) { await tp.file.rename(title); }

const area = await pickAreaHub();
const topics = await pickTopicTags();
const status = (await tp.system.suggester(
  ["seed (new, undeveloped)", "developing", "evergreen"],
  ["seed", "developing", "evergreen"], false, "Maturity")) ?? "seed";
const sources = await pickSourceNotes();
const tagBlock = yamlList([area.tag, ...topics]);
const sourceField = yamlField("source-notes", sources.map(f => wikiLink(f)));

/* Write the reciprocal link back into each Source's `ideas-extracted`. */
if (sources.length) {
  const ideaFile = tp.config.target_file;
  const ideaLink = `[[${ideaFile.path.replace(/\.md$/, "")}|${title}]]`;
  const integrate = await tp.system.suggester(
    ["No — source still has more to give", "Yes — set reading_status: integrated"],
    ["no", "yes"], false, `Mark ${sources.length} source note(s) as integrated?`);
  for (const f of sources) {
    await app.fileManager.processFrontMatter(f, fm => {
      const list = Array.isArray(fm["ideas-extracted"]) ? fm["ideas-extracted"] : [];
      if (!list.includes(ideaLink)) list.push(ideaLink);
      fm["ideas-extracted"] = list;
      if (integrate === "yes") fm["reading_status"] = "integrated";
    });
  }
  new Notice(`Linked idea to ${sources.length} source note(s)`);
}
-%>
---
Parent: "[[AREAS/<% area.hubName %>|Link]]"
Type: Idea
status: <% status %>
<% sourceField %>
tags:
<% tagBlock %>
---
# <% title %>


---
## Related notes (shared topics)
```dataviewjs
const here = dv.current();
const myTopics = Array.from(here.file.etags ?? []).map(String).filter(t => t.startsWith("#topic/"));
if (myTopics.length === 0) {
  dv.paragraph("*No topic tags on this note yet.*");
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
