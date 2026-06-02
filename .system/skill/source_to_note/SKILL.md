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
9. Link related notes where applicable.
10. Assign a confidence level to each note.
11. Verify that each note filename matches the slugified title. If not, rename the file.
12. Set metadata status to `publish`.

## Atomic Note Rules

See the full atomic note rules in the implementation plan.

Key rules:
- One reusable idea per note.
- Each note must be understandable without reading the full source.
- Prefer fewer high-quality notes over many shallow notes.
- Do not force-split. A single source should typically produce one note unless topics are truly too diverse.
- Titles must be descriptive. Avoid single-word titles unless the word is a recognized proper name. Less than 18 words; aim for fewer, not more.

## Filename Rules

- Generated notes must use lowercase kebab-case filenames.
- Notes must use `.md` extension.
- Note filenames must match note titles. Derive the filename by slugifying the title to lowercase kebab-case.

Example: A note titled "How Transformer Attention Works" becomes `how-transformer-attention-works.md`.

## Guiding Principle

Create knowledge, not summaries.

## Template

All generated notes must follow `.system/templates/note.md`.

## Output

Knowledge notes in `notes/` and updated metadata sidecar files.
