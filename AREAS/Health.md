---
Parent: "[[Index|Link]]"
Type: Index
tags:
  - area/health
---
# Health

## Projects
```dataview
TABLE Project AS "Project", join(filter(file.etags, (t) => startswith(t, "#topic/")), ", ") AS "Topics"
FROM "PROJECTS"
WHERE file.name = "00000" AND contains(file.tags, "#area/health")
```

## Ideas
```dataview
TABLE status AS "Status"
WHERE Type = "Idea" AND contains(file.tags, "#area/health")
SORT status ASC
```

## Resources
```dataview
LIST
FROM "RESOURCES"
WHERE contains(file.tags, "#area/health")
```

## Sources
```dataview
LIST
WHERE Type = "Source" AND contains(file.tags, "#area/health")
```
