<%*
// New Source note — lands in Inbox/, classification deferred to the Idea note.
const title = await tp.system.prompt("Source title");
const url    = await tp.system.prompt("URL or PDF path", "");
if (tp.file.title.startsWith("Untitled")) {
  await tp.file.rename(title);
}
-%>
---
Type: Source
Parent: "[[Inbox/00000|Link]]"
reading_status: inbox
source: <% url %>
author:
ideas-extracted: []
tags: []
---

# <% title %>

## Highlights
<%* /* ==highlights== (web) or pdf-plus backlinks (papers) land here while reading */ -%>

## Digest — owes an Idea note
- [ ] Cluster highlights by concept (not by the source's chapter order)
- [ ] Write Idea note(s) in your own words; backlink this Source
- [ ] Add the Idea link to `ideas-extracted`, set `reading_status: integrated`
