---
Parent: "[[Index|Link]]"
Type: Index
---
# Boards

Per-list task tables (Backlog / In Progress, with clickable task notes) live on [[Index]].

## All kanban boards
```dataview
TABLE Project AS "Project", join(filter(file.etags, (t) => startswith(t, "#area/")), ", ") AS "Area"
FROM "PROJECTS"
WHERE Type = "Tasks"
```
