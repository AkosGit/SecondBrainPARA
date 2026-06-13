<%* 
let title = tp.file.title 
let sSubject = ""
let sCategory = ""

if (title.startsWith("Untitled")) { 
  title = "00000"
} await tp.file.rename(title);

sArea = tp.file.folder()

setTimeout(() => {
  app.fileManager.processFrontMatter(tp.config.target_file, frontmatter => {
  frontmatter["Parent"] = `[[Index|Link]]`
  frontmatter["Type"] = "Area"
  frontmatter["Area"] = sArea
  })
}, 200)
-%>
# <% sArea %>

## Projects
```dataview
TABLE Project AS "Project" 
FROM "PROJECTS" 
WHERE file.name = "00000" AND Area = "<% tp.file.folder() %>"
```

## Ideas
```dataview
LIST 
WHERE contains(file.folder, this.file.folder) AND Type = "Idea"
```

## Resources
```dataview
LIST 
FROM "RESOURCES"
WHERE Area = "<% tp.file.folder() %>"
```
