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

async function pickTopicTags() {
  const chosen = [];
  while (true) {
    const existing = allFrontmatterTags("topic/").filter(t => !chosen.includes(t));
    const labels = [...existing, "New topic...", "Done (" + chosen.length + " selected)"];
    const values = [...existing, "__new__", "__done__"];
    const pick = await tp.system.suggester(labels, values, false, "Extra topic tags (optional)");
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

setTimeout(async () => {
  const folder = tp.file.folder();
  const idx = await tp.file.find_tfile(`${folder}/00000.md`);
  const idxFm = app.metadataCache.getFileCache(idx)?.frontmatter ?? {};
  const inherited = (idxFm.tags ?? []).map(String);

  const noteType = await tp.system.suggester(
    ["Resource", "Idea", "Source"],
    ["Resource", "Idea", "Source"],
    false, "Note type");

  const extraTopics = await pickTopicTags();
  const tags = Array.from(new Set([...inherited, ...extraTopics]));

  await app.fileManager.processFrontMatter(tp.config.target_file, fm => {
    fm["Parent"] = `[[PROJECTS/${folder}/00000|Link]]`;
    fm["Type"] = noteType;
    if (noteType === "Idea") {
      fm["status"] = "seed";
      fm["source-notes"] = [];
    }
    fm["Project"] = folder;
    fm["tags"] = tags;
  });
}, 200);
-%>
# <% title %>
