<%* 
let title = tp.file.title;

if (title.startsWith("Untitled")) { 
  title = await tp.system.prompt("Title"); 
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

async function getCategory() {
  // Get all files in the specified folder
	const folder = tp.file.folder();
	const filePath = `${folder}/00000.md`;
	const file = await tp.file.find_tfile(`${tp.file.folder()}/00000.md`);
	const metadata = app.metadataCache.getFileCache(file)?.frontmatter;
	return metadata.Category
}


async function getSubCategoryOptions() {
  // Get all files in the specified folder
  const files = app.vault.getFiles()

  // Extract unique values for the `Area` property in frontmatter
  const subcategorys = new Set();
  for (const file of files) {
    const metadata = app.metadataCache.getFileCache(file)?.frontmatter;
    if (metadata && metadata.Subcategory) {
      subcategorys.add(metadata.Subcategory);
    }
  }

  // Convert the Set to an array and sort it
 let options = Array.from(subcategorys).sort();
 options.push("Other...")
 

  // Prompt the user with a dropdown menu
  const selection = await tp.system.suggester(options, options);
  let finalanswer = selection
  if(selection == "Other..."){
	finalanswer = await tp.system.prompt("New subcategory:");
  }
  return finalanswer;
}


// Ensure async behavior by awaiting the results
setTimeout(async () => {
  const areaSelection = await getArea()
  const categorySelection = await getCategory();
  const subcategorySelection = await getSubCategoryOptions();
  app.fileManager.processFrontMatter(tp.config.target_file, frontmatter => {
    frontmatter["Parent"] = `[[PROJECTS/${tp.file.folder()}/00000|Link]]`
    frontmatter["Type"] = "ProjectFile";
    frontmatter["Area"] = areaSelection;
    frontmatter["Category"] = categorySelection
    frontmatter["Subcategory"] = subcategorySelection
    frontmatter["Project"] = tp.file.folder();
  });
}, 200);

-%>
# <% title %>
