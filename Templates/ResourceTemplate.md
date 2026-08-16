<%*
/* A Resource is a FINISHED ARTIFACT (SYSTEM.md §1.2).

   This template is location-aware, so it does the right thing wherever you run it:
   - inside PROJECTS/<X>/ or ARCHIVE/<X>/ → a project deliverable. Parent → the
     project's 00000, stamps `Project:`, inherits the project index's tags, and
     offers a board card. Identical result to ProjectFileTemplate → Resource.
   - anywhere else → a standalone artifact. Moves the note to RESOURCES/ and
     asks for area + topics.
   ⚠️ `addBoardCard()` is duplicated from `ProjectFileTemplate.md` — keep in sync. */
function slugify(s) {
  return String(s).trim().toLowerCase().replace(/[^a-z0-9/]+/g, "-").replace(/^-+|-+$/g, "");
}
function yamlList(items) {
  return items.length ? items.map(t => "  - " + t).join("\n") : "  []";
}
async function addBoardCard(folderPath, noteFile, noteTitle) {
  const list = await tp.system.suggester(
    ["Backlog", "In Progress", "No card"], ["Backlog", "In Progress", null],
    false, "Add a card to the project board?");
  if (!list) return;
  const inFolder = app.vault.getMarkdownFiles().filter(f => f.parent?.path === folderPath);
  const board = inFolder.find(f => {
    const fm = app.metadataCache.getFileCache(f)?.frontmatter;
    return fm?.Type === "Tasks" || fm?.["kanban-plugin"] === "board";
  }) ?? inFolder.find(f => /\bTasks$/i.test(f.basename));
  if (!board) {
    new Notice(`No board found in ${folderPath} — card skipped. Add one with TasksTemplate.`, 8000);
    return;
  }
  const lines = (await app.vault.read(board)).split("\n");
  const start = lines.findIndex(l => l.trim() === "## " + list);
  if (start === -1) { new Notice(`No "${list}" list on the board — card skipped.`, 8000); return; }
  let end = start + 1;
  while (end < lines.length
      && !/^##\s/.test(lines[end])
      && !/^%%\s*kanban:settings/.test(lines[end])) end++;
  let last = end - 1;
  while (last > start && lines[last].trim() === "") last--;
  lines.splice(last + 1, 0, `- [ ] [[${noteFile.path.replace(/\.md$/, "")}|${noteTitle}]]`);
  await app.vault.modify(board, lines.join("\n"));
  new Notice(`Added to ${list}: ${noteTitle}`);
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
  title = (await tp.system.prompt("Title")) ?? title;
}

const startPath = tp.config.target_file.path;
const inProject = startPath.startsWith("PROJECTS/") || startPath.startsWith("ARCHIVE/");
const folderPath = tp.file.folder(true);
const folderName = tp.file.folder();

let parentLink, extraFields, tagBlock;

if (inProject) {
  if (title !== tp.file.title) { await tp.file.rename(title); }
  const idx = app.vault.getAbstractFileByPath(`${folderPath}/00000.md`);
  const idxTags = (idx ? (app.metadataCache.getFileCache(idx)?.frontmatter?.tags ?? []) : []).map(String);
  const extraTopics = await pickTopicTags();
  tagBlock = yamlList(Array.from(new Set([...idxTags, ...extraTopics])));
  parentLink = `${folderPath}/00000`;
  extraFields = `Project: "${folderName}"\n`;
  await addBoardCard(folderPath, tp.config.target_file, title);
} else {
  if (!startPath.startsWith("RESOURCES/")) {
    await tp.file.move("RESOURCES/" + title);
  } else if (title !== tp.file.title) {
    await tp.file.rename(title);
  }
  const area = await pickAreaHub();
  const topics = await pickTopicTags();
  tagBlock = yamlList([area.tag, ...topics]);
  parentLink = `AREAS/${area.hubName}`;
  extraFields = "";
}
-%>
---
Parent: "[[<% parentLink %>|Link]]"
Type: Resource
<% extraFields %>tags:
<% tagBlock %>
---
# <% title %>
