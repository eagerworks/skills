# AGENTS.md

## Markdown line wrapping

Never hard-wrap prose inside a markdown paragraph or list item — write each one as a single logical line (however long) and let the viewing tool (editor, GitHub diff view, terminal) soft-wrap it for display. Hard-wrapping at a fixed column bakes a stale width into the source: editing one sentence in the middle of a hard-wrapped paragraph reflows every line after it, turning a one-sentence change into a noisy multi-line diff. Code fences, tables, and headings are unaffected — this applies to prose only.

`npx prettier --prose-wrap=never --embedded-language-formatting=off --write <file>.md` reflows an already hard-wrapped file back to one line per paragraph/list item without touching fenced code blocks.
