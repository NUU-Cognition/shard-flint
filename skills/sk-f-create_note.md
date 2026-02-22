This skill belongs to the Flint shard. Ensure you have [[init-f]] in context before continuing.

# Skill: Create Concept Note

Create an evergreen concept note — an atomic, concept-oriented note designed to accumulate value over time through dense linking.

# Input

- A concept, insight, or idea to capture (from conversation, research, or user request)

# Actions

1. **Identify the concept.** Distill the input down to a single atomic concept. If there are multiple ideas, create multiple notes. Ask: "What is the one thing this note is about?"

2. **Craft the title.** The title IS the concept — a complete phrase that stands alone. Concept-oriented, not source-oriented.
   - Good: "Context compaction degrades protocol adherence", "Evergreen notes accumulate value through connections"
   - Bad: "Notes on context windows", "Meeting insights Dec 2025"

3. **Check for existing notes.** Search `Mesh/Notes/` for notes that already cover this concept. If one exists, update it instead of creating a duplicate.

4. **Create the note.** Follow [[tmp-f-note]] template.
   - Place in `Mesh/Notes/`
   - Generate UUID for the `id` field
   - Opening paragraph: self-contained assertion or definition (1-3 sentences)
   - Elaboration: evidence, implications, nuance
   - Related: link to every connected concept note and artifact

5. **Dense linking.** This is the most important step.
   - Search the Mesh for related concept notes, tasks, plans, notepads — anything connected
   - Add `[[wikilinks]]` in the Related section of the new note
   - Update each related note's Related section to link back to the new note (bidirectional)
   - Prefer fine-grained associative links over broad categorical ones

6. **Report.** Tell the user what was created and what was linked.

# Output

- New concept note in `Mesh/Notes/`
- Updated Related sections in linked notes (bidirectional)
