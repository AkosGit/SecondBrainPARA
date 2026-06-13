
## [[Boards]]

## Areas
```dataview
TABLE Area as "Area"
FROM "AREAS" 
WHERE file.name = "00000"
```

## Projects
```dataview
TABLE Project as "Project", Area AS "Area"
FROM "PROJECTS"
WHERE file.name = "00000"
```

## Resources
```dataview
TABLE Area 
FROM "RESOURCES"
```

## Archived projects
```dataview
TABLE Project as "Project", Area AS "Area"
FROM "ARCHIVE"
WHERE file.name = "00000"
```

## Categories and subcategories
```dataviewjs

const notes = await dv.query (`
TABLE 
  R.file.link as Note, R.Subcategory as Subcategory
GROUP BY
  Category
FLATTEN rows as R
SORT Category, R.Subcategory
`)

console.log(notes)

if (!notes.successful) {
  dv.paragraph(`~~~~\n${ notes.error }\n~~~~\n`)
  return
}

let typeDict = {}
for (let note of notes.value.values) {
  if ( !typeDict.hasOwnProperty(note[0]) )
    typeDict[note[0]] = []

  typeDict[note[0]].push([...note.slice(1)])
}

for (let key of Object.keys(typeDict)) {
  dv.header(2, key)
  dv.table([...notes.value.headers.slice(1)],
    typeDict[key])
}
```

## Inactive projects
```dataview
TABLE Project, Area
FROM "ARCHIVE"
```

## Files without parent links
```dataview
table file.name as "Note Name"
from ""
where !Parent and !contains(file.name, "Template")  and file.name != "Index"
sort file.name asc
```


## Sync conflicted files
```dataview
TABLE file.name AS "File", file.path AS "Path"
FROM ""
WHERE contains(file.name, ".sync-conflict")
SORT file.name ASC
```