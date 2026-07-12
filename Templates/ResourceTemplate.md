<%*
let title = tp.file.title;
if (title.startsWith("Untitled")) {
  title = await tp.system.prompt("Title");
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
    const pick = await tp.system.suggester(labels, values, false, "Topic tags (optional)");
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
  fm["Type"] = "Resource";
  fm["tags"] = [area.tag, ...topics];
});
-%>
# <% title %>
