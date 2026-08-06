---
Parent: "[[AREAS/Health|Link]]"
Type: Idea
status: seed
source-notes:
  - "[[Inbox/test file|test file]]"
tags:
  - area/health
  - topic/health-problems
  - topic/wd
---
# test idea


---
## Related notes (shared topics)
```dataviewjs
const here = dv.current();
const myTopics = Array.from(here.file.etags ?? []).map(String).filter(t => t.startsWith("#topic/"));
if (myTopics.length === 0) {
  dv.paragraph("*No topic tags on this note yet.*");
} else {
  const related = dv.pages().where(p =>
    p.file.path !== here.file.path &&
    Array.from(p.file.etags ?? []).map(String).some(t => myTopics.includes(t)));
  if (related.length === 0) {
    dv.paragraph("*Nothing shares a topic with this note yet.*");
  } else {
    dv.table(["Note", "Type"], related.map(p => [p.file.link, p.Type]));
  }
}
```
