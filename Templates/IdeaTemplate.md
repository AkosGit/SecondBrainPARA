<%* 
let title = tp.file.title 

if (title.startsWith("Untitled")) { 
  title = await tp.system.prompt("Title"); 
} await tp.file.rename(title);


setTimeout(() => {
  app.fileManager.processFrontMatter(tp.config.target_file, frontmatter => {
  frontmatter["Parent"] = `[[AREAS/${tp.file.folder()}/00000|Link]]`
  frontmatter["Type"] = "Idea"
  frontmatter["Area"] = tp.file.folder()
  frontmatter["tags"] = []
  })
}, 200)
-%>
# <% title %>
