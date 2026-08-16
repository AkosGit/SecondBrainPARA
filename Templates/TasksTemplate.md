<%*
/* Manual path: adds a board to a project that doesn't have one. New projects
   get their board automatically from `ProjectIndexTemplate`, which builds the
   same skeleton inline (it can't call this template — see the comment there).

   ⚠️ KEEP IN SYNC with the `createBoard()` function in `ProjectIndexTemplate.md`. */
const folderPath = tp.file.folder(true);   // e.g. "PROJECTS/Fatigue"
const folderName = tp.file.folder();       // e.g. "Fatigue"

let title = tp.file.title;
if (title.startsWith("Untitled")) { title = folderName + " Tasks"; }
if (title !== tp.file.title) { await tp.file.rename(title); }

const idx = app.vault.getAbstractFileByPath(`${folderPath}/00000.md`);
const areaTags = (idx ? (app.metadataCache.getFileCache(idx)?.frontmatter?.tags ?? []) : [])
  .map(String).filter(t => t.startsWith("area/"));
const tagBlock = areaTags.length ? areaTags.map(t => "  - " + t).join("\n") : "  []";
-%>
---
Parent: "[[<% folderPath %>/00000|Link]]"
Type: Tasks
Project: "<% folderName %>"
kanban-plugin: board
tags:
<% tagBlock %>
---

## Backlog

## In Progress

## Done

%% kanban:settings
```
{"kanban-plugin":"board","list-collapse":[false,false,false],"show-checkboxes":false}
```
%%
