---
description: "Construct a stitch file — a curated index of [[wikilinks]] to feed into `flint helper stitch`"
---

Run `flint shard start-dev f` if you haven't already.

# Skill: Stitch

Construct a **stitch file** — a markdown file that bundles together everything the user wants stitched into a single exportable context. The file's body is a curated list of `[[wikilinks]]` to Mesh artifacts. Once created, the file is fed into `flint helper stitch` to resolve every link and copy the combined content to the clipboard for export to another LLM, document, or chat.

This skill does NOT run `flint helper stitch` itself. It builds the input file and emits the exact command the user can paste into their terminal.

# Input

- A topic, theme, or set of artifacts to bundle (e.g. "everything about the auth rewrite", "the last three reports", "specs + tasks for the shard registry")
- (Optional) Explicit list of artifacts to include — if provided, use these verbatim and skip the search step
- (Optional) Target location — defaults to `Mesh/Notes/`
- (Optional) Stitch file title — defaults to a sensible name derived from the topic

# Actions

1. **Gather artifacts.** Unless the user gave an explicit list:
   - Search `Mesh/` for artifacts related to the topic — by filename, tags, frontmatter fields, and body content
   - Cast wide: typed artifacts (`(Task)`, `(Report)`, `(Notepad)`, `(Spec)`, `(Plan)`, etc.), notes, dashboards, system files, and metadata references
   - Deduplicate; preserve a useful order (e.g. by type, then by number)

2. **Present and confirm.** Show the user:
   - The proposed stitch file title and target path
   - The full list of wikilinks that will be included, grouped by type
   - A short description (one or two sentences) that will sit at the top of the file
   - Ask the user to confirm, edit, add, or remove entries before writing

3. **Choose the path.**
   - Default: `Mesh/Notes/(Stitch) <Title>.md`
   - If the user specified a folder, write there; if they specified a different prefix or untyped name, honor it
   - Use a filename that will be unique inside the chosen folder

4. **Write the stitch file.** Create the file with this structure:

   ```markdown
   ---
   id: [generate-uuid]
   tags:
     - "#note"
     - "#note/stitch"
   orbh-sessions:
     - "[[your-session-id]]"
   authors:
     - "[[@Person Name]]"  /* from .flint/identity.json; omit if no identity set */
   ---

   # [Stitch Title]

   [one or two sentence description of what this bundles and why]

   ## Sources

   - [[Artifact 1]]
   - [[Artifact 2]]
   - (continue)
   ```

   Notes on the body:
   - Anything that is NOT a `[[wikilink]]` is preserved verbatim in the stitched output as the leading section. Use this space to add context that the receiving LLM/reader should know about the bundle.
   - Wikilinks may appear anywhere — `flint helper stitch` extracts them from the entire body. Grouping under headings (e.g. `## Tasks`, `## Reports`) is purely cosmetic.
   - Avoid putting example `[[Foo]]` wikilinks inside backticks if you do NOT want them resolved — `flint helper stitch` strips code spans before parsing, so backticked examples are correctly ignored. Use this to your advantage when documenting.

5. **Emit the command.** Print the exact terminal command the user can copy-paste, quoting the path to handle spaces and parens:

   ```
   flint helper stitch "Mesh/Notes/(Stitch) <Title>.md"
   ```

   Also offer the `--print` variant for piping (`flint helper stitch --print "<path>" | pbcopy`-style workflows) and `--also-print` for both-clipboard-and-stdout.

6. **Open in Obsidian** (optional — only if helpful in context). Use the platform-appropriate `obsidian://open?vault=...&file=...` URL so the user can review the file before stitching.

# Output

- A stitch file written to the chosen path (default `Mesh/Notes/(Stitch) <Title>.md`)
- The exact `flint helper stitch "<path>"` command printed for the user to paste into their terminal

# Notes

- The skill does not invoke `flint helper stitch`. The user runs it themselves so the clipboard ends up on their machine, not the agent's.
- A stitch file is a regular note. It can be edited, re-stitched, renamed, or deleted via the standard helpers (`flint helper rename`, `flint helper delete`).
- `flint helper stitch` is one-level only — it does not recurse into wikilinks-of-wikilinks. If a bundle needs to flatten a deep graph, expand it manually in the stitch file before running the command.
