---
Type: Index
---
# Index

*New here? [[How to Use This Vault — Walkthrough]] · [[Capture — Setup and Usage]] · [[SYSTEM]]*

## Tasks

### In Progress
```dataview
TABLE WITHOUT ID
  choice(length(Tasks.outlinks) > 0, Tasks.outlinks[0], split(Tasks.text, " @")[0]) AS "Task",
  regexreplace(Tasks.text, "[^0-9-]", "") AS "Due date",
  file.link AS "Board"
FROM "PROJECTS"
FLATTEN file.tasks AS Tasks
WHERE contains(meta(Tasks.section).subpath, "In Progress")
SORT regexreplace(Tasks.text, "[^0-9-]", "") ASC
```

### Backlog
```dataview
TABLE WITHOUT ID
  choice(length(Tasks.outlinks) > 0, Tasks.outlinks[0], split(Tasks.text, " @")[0]) AS "Task",
  regexreplace(Tasks.text, "[^0-9-]", "") AS "Due date",
  file.link AS "Board"
FROM "PROJECTS"
FLATTEN file.tasks AS Tasks
WHERE contains(meta(Tasks.section).subpath, "Backlog")
SORT regexreplace(Tasks.text, "[^0-9-]", "") ASC
```

All boards: [[Boards]]

## Projects by area
```dataviewjs
const projects = dv.pages('"PROJECTS"').where(p => p.file.name === "00000");
const groups = {};
for (const p of projects) {
  const tags = Array.from(p.file.etags ?? []).map(String);
  const area = tags.find(t => t.startsWith("#area/")) ?? "#area/unsorted";
  (groups[area] ??= []).push(p);
}
const keys = Object.keys(groups).sort();
if (keys.length === 0) { dv.paragraph("*No projects yet.*"); }
for (const key of keys) {
  dv.header(3, key.replace("#area/", ""));
  dv.table(["Project", "Topics"], groups[key].map(p =>
    [p.file.link, Array.from(p.file.etags ?? []).filter(t => String(t).startsWith("#topic/")).join(", ")]));
}
```

## Areas
```dataview
TABLE join(filter(file.etags, (t) => startswith(t, "#area/")), ", ") AS "Tag"
FROM "AREAS"
WHERE Type = "Index"
```

## Reading queue

```dataviewjs
const inbox = dv.pages('"Inbox"').where(p => p.Type === "Source" && p.reading_status !== "integrated");
const n = inbox.length;
if (n > 10) {
  dv.paragraph(`⚠️ **${n} sources in the inbox — digest or delete before capturing more.** (cap: 10)`);
} else {
  dv.paragraph(`${n} source(s) awaiting digest. (cap: 10)`);
}
```

### Inbox / reading
```dataview
TABLE WITHOUT ID file.link AS "Source", reading_status AS "Status", (date(today) - file.cday).day AS "Age (days)"
FROM "Inbox"
WHERE Type = "Source" AND reading_status != "integrated"
SORT file.ctime ASC
```

### Stale (older than 14 days)
```dataview
TABLE WITHOUT ID file.link AS "Source", (date(today) - file.cday).day AS "Age (days)"
FROM "Inbox"
WHERE Type = "Source" AND reading_status != "integrated" AND (date(today) - file.cday).day > 14
SORT file.ctime ASC
```

### Integrated
```dataview
LIST
FROM "Inbox"
WHERE Type = "Source" AND reading_status = "integrated"
```

## Ideas

### Seeds needing development
```dataview
TABLE status AS "Status", join(filter(file.etags, (t) => startswith(t, "#topic/")), ", ") AS "Topics"
FROM ""
WHERE Type = "Idea" AND (status = "seed" OR status = "developing")
SORT status ASC
```

### Serendipity — three random ideas
```dataviewjs
const ideas = dv.pages().where(p => p.Type === "Idea").array();
if (ideas.length === 0) {
  dv.paragraph("*No ideas yet.*");
} else {
  const pool = [...ideas];
  const picks = [];
  while (picks.length < 3 && pool.length > 0) {
    picks.push(pool.splice(Math.floor(Math.random() * pool.length), 1)[0]);
  }
  dv.list(picks.map(p => p.file.link));
}
```

## Notes by topic
```dataviewjs
const pages = dv.pages().where(p =>
  Array.from(p.file.etags ?? []).filter(t => String(t).startsWith("#topic/")).length > 0);
const groups = {};
for (const p of pages) {
  for (const t of Array.from(p.file.etags ?? []).filter(t => String(t).startsWith("#topic/"))) {
    (groups[t] ??= []).push(p);
  }
}
const keys = Object.keys(groups).sort();
if (keys.length === 0) { dv.paragraph("*No topic-tagged notes yet.*"); }
for (const key of keys) {
  dv.header(3, String(key).replace("#topic/", ""));
  dv.table(["Note", "Type"], groups[key].map(p => [p.file.link, p.Type]));
}
```

## Resources
```dataview
TABLE join(filter(file.etags, (t) => startswith(t, "#topic/")), ", ") AS "Topics"
FROM "RESOURCES"
```

## Archived projects
```dataview
TABLE Project AS "Project", join(filter(file.etags, (t) => startswith(t, "#area/")), ", ") AS "Area"
FROM "ARCHIVE"
WHERE file.name = "00000"
```

## Audit

### Notes without a Parent link
```dataview
LIST
FROM "PROJECTS" OR "AREAS" OR "RESOURCES" OR "Inbox"
WHERE !Parent
```

### Project indexes without an area tag
```dataview
LIST
FROM "PROJECTS"
WHERE file.name = "00000" AND length(filter(file.etags, (t) => startswith(t, "#area/"))) = 0
```
