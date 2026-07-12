<%*
let title = tp.file.title;
if (title.startsWith("Untitled")) {
  title = await tp.system.prompt("Area name");
}
await tp.file.rename(title);

const slug = title.trim().toLowerCase().replace(/[^a-z0-9]+/g, "-").replace(/^-+|-+$/g, "");
const areaTag = "area/" + slug;

setTimeout(async () => {
  await app.fileManager.processFrontMatter(tp.config.target_file, fm => {
    fm["Parent"] = "[[Index|Link]]";
    fm["Type"] = "Index";
    fm["tags"] = [areaTag];
  });
}, 200);
-%>
# <% title %>

## Projects
```dataview
TABLE Project AS "Project", join(filter(file.etags, (t) => startswith(t, "#topic/")), ", ") AS "Topics"
FROM "PROJECTS"
WHERE file.name = "00000" AND contains(file.tags, "#<% areaTag %>")
```

## Ideas
```dataview
LIST
WHERE Type = "Idea" AND contains(file.tags, "#<% areaTag %>")
```

## Resources
```dataview
LIST
FROM "RESOURCES"
WHERE contains(file.tags, "#<% areaTag %>")
```

## Sources
```dataview
LIST
WHERE Type = "Source" AND contains(file.tags, "#<% areaTag %>")
```
