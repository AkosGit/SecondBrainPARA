<%* 
let title = tp.file.title;

if (title.startsWith("Untitled")) { 
  title = tp.file.folder(); 
  title = title + " Tasks"
} 
await tp.file.rename(title);

async function getArea() {
  // Get all files in the specified folder
	const folder = tp.file.folder();
	const filePath = `${folder}/00000.md`;
	console.log(filePath)
	const file = await tp.file.find_tfile(`${tp.file.folder()}/00000.md`);
	const metadata = app.metadataCache.getFileCache(file)?.frontmatter;
	return metadata.Area
}

// Ensure async behavior by awaiting the results
setTimeout(async () => {
  const areaSelection = await getArea() // Wait for getOptions to finish
  
  app.fileManager.processFrontMatter(tp.config.target_file, frontmatter => {
    frontmatter["Parent"] = `[[PROJECTS/${tp.file.folder()}/00000|Link]]`
    frontmatter["Type"] = "Tasks";
    frontmatter["Area"] = areaSelection;
    frontmatter["Project"] = tp.file.folder();
    frontmatter["kanban-plugin"] = "board"
  });
}, 200);

-%>

## Backlog

## In Progress

## Done

%% kanban:settings
```
{"kanban-plugin":"board","list-collapse":[false,false,false],"show-checkboxes":false}
```
%%