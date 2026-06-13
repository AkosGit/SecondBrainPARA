<%* 
let title = tp.file.title;

if (title.startsWith("Untitled")) { 
  title = "00000"; 
} 
await tp.file.rename(title);

async function getCategoryOptions() {
  // Get all files in the specified folder
  const files = app.vault.getFiles()

  // Extract unique values for the `Area` property in frontmatter
  const categorys = new Set();
  for (const file of files) {
    const metadata = app.metadataCache.getFileCache(file)?.frontmatter;
    if (metadata && metadata.Category) {
      categorys.add(metadata.Category);
    }
  }

  // Convert the Set to an array and sort it
 let options = Array.from(categorys).sort();
 options.push("Other...")
 

  // Prompt the user with a dropdown menu
  const selection = await tp.system.suggester(options, options);
  let finalanswer = selection
  if(selection == "Other..."){
	finalanswer = await tp.system.prompt("New category:");
  }
  return finalanswer;
}

async function getOptions(folderPath) {
  // Get all files in the specified folder
  const files = app.vault.getFiles().filter(file => file.path.startsWith(folderPath + "/"));

  // Extract unique values for the `Area` property in frontmatter
  const areaValues = new Set();
  for (const file of files) {
    const metadata = app.metadataCache.getFileCache(file)?.frontmatter;
    if (metadata && metadata.Area) {
      areaValues.add(metadata.Area);
    }
  }

  // Convert the Set to an array and sort it
  const options = Array.from(areaValues).sort();

  // Prompt the user with a dropdown menu
  const selection = await tp.system.suggester(options, options);
  return selection;
}

// Async function for frontmatter processing
async function updateFrontmatter() {
  const areaSelection = await getOptions("AREAS");  // Await the result of getOptions
  const projectFolder = tp.file.folder();  // Get the folder of the current file
  const categorySelection = await getCategoryOptions()
  await app.fileManager.processFrontMatter(tp.config.target_file, frontmatter => {
    frontmatter["Parent"] = `[[AREAS/${areaSelection}/00000|Link]]`
    frontmatter["Type"] = "Project";
    frontmatter["Area"] = areaSelection;  // Assign selected area
    frontmatter["Category"] = categorySelection;
    frontmatter["Project"] = tp.file.folder();  // Assign selected project
	frontmatter["tags"] = []
  });
}

// Call the update function
updateFrontmatter();

-%>
# <% tp.file.folder() %>

## Project files 
```dataview 
LIST 
WHERE contains(file.folder, this.file.folder) AND file.name != "00000"
```