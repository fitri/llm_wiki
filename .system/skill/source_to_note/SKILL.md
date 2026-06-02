# Source to Note Skill

## Purpose

Convert source material into atomic knowledge notes.

## Workflow

1. Read a source file from `source/` that has `status: new`.
2. Update metadata status to `digest`.
3. Extract and analyze content from the source file.
4. Determine knowledge density and decide note count.
5. Generate one or more knowledge notes following the atomic note rules.
6. Derive each note filename from its title by slugifying the title to lowercase kebab-case.
7. Assign categories and tags from the taxonomy indexes.
8. Generate a concise summary for each note.
9. **Perform the Concept Linking Pass** (see below).
10. Verify filename-title parity for every generated note — ensure the `slug` field matches the note filename (without `.md`).
11. Set metadata status to `publish`.

## Atomic Note Rules

A knowledge note should contain one reusable idea.

**Create a separate note when the source contains:**
- A distinct concept.
- A distinct process or workflow.
- A distinct decision or conclusion.
- A distinct rule, principle, or best practice.
- A distinct technical reference.
- A distinct troubleshooting pattern.
- A distinct comparison between ideas, tools, or methods.
- A distinct reusable question-and-answer pair.

**Do not create separate notes for:**
- Minor details.
- Repeated statements.
- Temporary observations.
- Personal commentary with no reusable value.
- Source summaries that do not introduce reusable knowledge.
- Content that only makes sense inside the original document.

Each note must be understandable without reading the full original source.

Each note should answer at least one of these questions:
1. What is this?
2. How does this work?
3. Why does this matter?
4. When should this be used?
5. What problem does this solve?
6. What should be remembered from this?

Prefer fewer high-quality reusable notes over many shallow notes. Do not force-split a single source into multiple notes. Only split when topics are truly too big and diverse to fit in one note.

Titles must be descriptive. Avoid single-word titles unless the word is a recognized proper name. Less than 18 words; aim for fewer, not more. The title alone should inform what the note is about.

## Filename Rules

- Generated notes must use lowercase kebab-case filenames.
- Notes must use `.md` extension.
- Note filenames must match note titles. Derive the filename by slugifying the title to lowercase kebab-case.

Example: A note titled "How Transformer Attention Works" becomes `how-transformer-attention-works.md`.

## Guiding Principle

Create knowledge, not summaries.

## Concept Linking Pass

After generating the note, scan it for important concepts and insert `[[wikilinks]]` inline.

**Concept Linking Rules:**

1. Scan the entire note and identify all important concepts.
2. Concepts include: technical terms, technologies, frameworks, tools, methodologies, architectures, standards, protocols, scientific concepts, business concepts, domain-specific terminology, important people, organizations, and products when relevant.
3. Convert identified concepts into `[[wikilinks]]` using lowercase kebab-case format.
4. If a concept appears as "transformer", create `[[transformer]]`, not `[[Transformer]]`.
5. If a concept appears as "risc-v", create `[[risc-v]]`, not `[[RISC-V]]`.
6. Link only meaningful concepts. Do not link common words, generic language, or filler terms.
7. Avoid excessive linking. Prioritize concepts that could reasonably become standalone notes.
8. Link the first meaningful occurrence of a concept. Additional occurrences should only be linked if it improves readability.
9. Preserve the original meaning, structure, and wording of the note.
10. Do not create a separate section listing concepts. Insert links directly into the note content.
11. Before finalizing, review the note and ensure all major concepts are linked.

**Example:**

Before:
`Transformer models use attention mechanisms. Embeddings are stored in vector databases. Retrieval augmented generation improves knowledge retrieval.`

After:
`[[transformer]] models use [[attention-mechanism]]s. [[embedding]]s are stored in [[vector-database]]s. [[retrieval-augmented-generation]] improves knowledge retrieval.`

## Auto Concept Note Creation

When a wikilink target does not correspond to an existing note in `notes/`, auto-create a stub:

1. Create `source/target-name.md` with content:
   ```markdown
   # Target Name

   ## Summary

   To be updated.
   ```
2. Create `source/target-name.md.metadata.json` with `type: auto`, `status: pending`.
3. Invoke the pipeline for this file to generate `notes/target-name.md` following `.system/templates/note.md` with placeholder content in all sections.
4. The auto note's `status` remains `pending` in both source metadata and note frontmatter.
5. Report the created stub back to the main agent.
6. Auto-created stubs use minimal but relevant categories and tags aligned with the concept title.

Auto stubs with `type: auto` and `status: pending` are exempt from the following checks:
- No 5-word minimum for naming
- Empty or placeholder content is allowed
- But template structure and slug parity must still be maintained

When a human later adds real content to a pending auto stub, the Metadata Agent picks it up on the next invocation (it scans for `status: pending` files with new content) and re-enters the pipeline at `status: new`.

## Template

All generated notes must follow `.system/templates/note.md`.

## Output

Knowledge notes in `notes/` and updated metadata sidecar files.
