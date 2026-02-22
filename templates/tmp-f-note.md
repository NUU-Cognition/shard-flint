# Filename: Mesh/Notes/[Concept Title].md

/* Concept notes are evergreen notes — written and organized to evolve, contribute, and accumulate over time, across projects.

   TITLE RULES:
   - The title IS the concept. It should be a complete phrase that can stand alone.
   - Concept-oriented, not source-oriented.
   - Good: "Atomic notes enable reusable arguments", "Context windows limit agent memory"
   - Bad: "Notes from meeting", "Chapter 3 thoughts", "Ideas about X"
   - Think of titles like APIs — they should communicate the concept without needing to read the body.

   These notes do NOT get a (Type) prefix. They are untyped — just the concept title as the filename.
*/

```markdown
---
id: [generate-uuid4]
tags:
  - "#note"
[agent]-sessions: /* replace [agent] with agent type (claude, codex, etc.) */
template: tmp-f-note
---

[Core assertion or concept definition in 1-3 sentences. This is the atomic unit — one concept, captured completely. The opening should be self-contained: if someone reads only this paragraph, they should understand the idea.]

[Elaboration, evidence, implications, or nuance. Develop the concept further. This section grows over time as understanding deepens. Write for yourself — clarity over polish.]

(continue)

## Related

- [[concept notes, artifacts, or any documents that connect to this idea]]
- (continue)

/* Related links are the most important part of an evergreen note. Every note should link to related concepts. The value of the note network grows with its connections, not its individual entries. Prefer fine-grained associative links over broad categorical ones. */
```

/* EVERGREEN NOTE PRINCIPLES (for the agent creating and maintaining these):

   1. ATOMIC — One concept per note. If you find yourself writing about two distinct ideas, make two notes and link them. A note should "capture the entirety of that thing" while addressing only one thing.

   2. CONCEPT-ORIENTED — Organize by concept, not by source (book, meeting, project). Ask: "In which context will I want to stumble upon this again?" not "Where does this come from?"

   3. DENSELY LINKED — Every note should link to related concepts via [[wikilinks]]. Links are more valuable than content. The connection network IS the knowledge structure. Backlinks implicitly define emerging concepts.

   4. ASSOCIATIVE — Don't force hierarchy or categories. Let structure emerge from connections. Prefer associative ontologies to hierarchical taxonomies. Items naturally belong in multiple contexts.

   5. PERSONAL — Write for yourself by default, disregarding audience. Optimizing for external readers during note-taking creates unnecessary overhead and writing paralysis. Opt into audience-focused writing explicitly.

   6. EVOLVING — These notes are living documents. Revisit, refine, and extend them over time. Better notes aren't about better writing — they're about better thinking.

   When creating a note, the agent should:
   - Check if an existing note already covers this concept (search Mesh/Notes/)
   - If so, update the existing note rather than creating a duplicate
   - Link the new note to every related concept note that exists
   - Update those related notes to link back (bidirectional linking)
*/
