<%*
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
