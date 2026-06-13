<%* 
let title = tp.file.title;

if (title.startsWith("Untitled")) { 
  title = await tp.system.prompt("Title"); 
} 
await tp.file.rename(title);
  
async function getArea(folderPath) {
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
 let options = Array.from(subcategorys);
 options.push("Other...")
 

  // Prompt the user with a dropdown menu
  const selection = await tp.system.suggester(options, options);
  let finalanswer = selection
  if(selection == "Other..."){
	finalanswer = await tp.system.prompt("New subcategory:");
  }
  return finalanswer;
}

async function updateFrontmatter() {
  const areaSelection = await getArea("AREAS");
  const categorySelection = await getCategoryOptions()
  const subcategorySelection = await getSubCategoryOptions()
  await app.fileManager.processFrontMatter(tp.config.target_file, frontmatter => {
  frontmatter["Parent"] = `[[AREAS/${areaSelection}/00000|Link]]`
  frontmatter["Type"] = "Resource"
  frontmatter["Area"] = areaSelection
  frontmatter["Category"] = categorySelection;
  frontmatter["Subcategory"] = subcategorySelection;
  frontmatter["tags"] = []
  })
}
updateFrontmatter()
-%>
# <% title %>
