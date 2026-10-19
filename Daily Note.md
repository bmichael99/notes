---
work_location:
:(:
:):
substances:
lunch:
---
# {{date:dddd, MMMM Do YYYY}}

<%*
const today = tp.file.title;
const yesterday = tp.date.now("YYYY-MM-DD", -1, today, "YYYY-MM-DD");

const priorNotes = app.vault.getMarkdownFiles()
  .filter((f) => f.path.startsWith("Daily Note/") && /^\d{4}-\d{2}-\d{2}$/.test(f.basename) && f.basename < today)
  .sort((a, b) => b.basename.localeCompare(a.basename));

const sourceDate = priorNotes.length ? priorNotes[0].basename : yesterday;
const linkLabel = sourceDate === yesterday ? "yesterday" : tp.date.now("MMMM Do", 0, sourceDate, "YYYY-MM-DD");

tR += `## Goals from [[${sourceDate}|${linkLabel}]]\n\n`;

const file = tp.file.find_tfile(sourceDate);
let body = "- [ ] ";
if (file) {
  const lines = (await app.vault.read(file)).split("\n");
  const start = lines.findIndex((line) => /^#{1,6}\s+Goals for tomorrow\s*$/i.test(line));
  if (start !== -1) {
    const chunk = [];
    for (let i = start + 1; i < lines.length; i++) {
      const line = lines[i];
      if (/^#{1,6}\s+/.test(line)) break;
      if (/^\d{4}-\d{2}-\d{2}\s/.test(line)) break;
      chunk.push(line);
    }
    while (chunk.length && /^\s*$/.test(chunk[0])) chunk.shift();
    while (chunk.length && /^(\s*|-\s*|-\s*\[[ xX]\]\s*)$/.test(chunk[chunk.length - 1])) chunk.pop();
    if (chunk.length) body = chunk.join("\n");
  }
}
tR += body;
%>

## Goals for tomorrow

- [ ] 

{{date}} {{time}} / w{{date:W}}
## Tags
 #graph-hidden 
# Thoughts