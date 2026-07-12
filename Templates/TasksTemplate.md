<%*
let title = tp.file.title;
if (title.startsWith("Untitled")) {
  title = tp.file.folder() + " Tasks";
}
await tp.file.rename(title);

setTimeout(async () => {
  const folder = tp.file.folder();
  const idx = await tp.file.find_tfile(`${folder}/00000.md`);
  const idxFm = app.metadataCache.getFileCache(idx)?.frontmatter ?? {};
  const inherited = (idxFm.tags ?? []).map(String).filter(t => t.startsWith("area/"));

  app.fileManager.processFrontMatter(tp.config.target_file, fm => {
    fm["Parent"] = `[[PROJECTS/${folder}/00000|Link]]`;
    fm["Type"] = "Tasks";
    fm["Project"] = folder;
    fm["tags"] = inherited;
    fm["kanban-plugin"] = "board";
  });
}, 200);
-%>

## Backlog

## In Progress

## Done

%% kanban:settings
```
{"kanban-plugin":"board","list-collapse":[false,false,false],"show-checkboxes":false}
```
%%
