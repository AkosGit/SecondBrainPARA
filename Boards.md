## In progress tasks

```dataview
TABLE WITHOUT ID split(Tasks.text, "@{")[0] as Task, 
regexreplace(Tasks.text, "[^0-9-]", "") AS "Due Date",
meta(Tasks.section).subpath as "Status", file.link as "Board" 
from "" 
FLATTEN file.tasks As Tasks 
WHERE contains(meta(Tasks.section).subpath,"In Progress") SORT date(Tasks.due.file.name)
```
## Kanban boards
```dataview
TABLE Project as "Project"
FROM "PROJECTS"
WHERE contains(file.name, " Tasks")
```

## Backlog tasks

```dataview
TABLE WITHOUT ID split(Tasks.text, "@{")[0] as Task, 
regexreplace(Tasks.text, "[^0-9-]", "") AS "Due Date",
meta(Tasks.section).subpath as "Status", file.link as "Board" 
from "" 
FLATTEN file.tasks As Tasks 
WHERE contains(meta(Tasks.section).subpath,"Backlog") SORT date(Tasks.due.file.name)
```