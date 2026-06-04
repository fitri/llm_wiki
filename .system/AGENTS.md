# Obsidian Knowledge Warehouse - Agent Instructions

## Pipeline Overview

The vault processes source material through a sequential pipeline:

```text
Human drops file into source/
    │
    ▼
Metadata Agent — scans source/, normalizes filenames, generates sidecars
    │
    ▼
filename.ext.metadata.json  (status: new)
    │
    ▼
Note Generation Agent — extracts content, generates atomic notes in notes/
    │
    ▼
notes/*.md  (status: publish)
    │
    ▼
Taxonomy Agent — manages controlled vocabularies in notes/categories.index.json and notes/tags.index.json
    │
    ▼
Health Check Agent — validates metadata, links, taxonomy, schema, filename-title parity
    │
    ▼
Vault is healthy → search + metadata + AI retrieval
```

There is a temporary workspace at `.digest/` for intermediate artifacts (OCR output, extracted text, processing artifacts). No permanent knowledge is stored there.

---

## Vault Structure

```text
vault/
├── source/          Human ingestion zone — drop files here
├── .digest/         Temporary AI workspace (OCR, extracted text, intermediates)
├── notes/           Final knowledge repository — flat, no subfolders
├── assets/          Images, diagrams, attachments
├── archive/         Cold storage for deprecated/retired notes
├── AGENTS.md        (symlink → .system/AGENTS.md)
└── .system/
    ├── templates/
    │   ├── note.md
    │   ├── file.metadata.json
    │   ├── categories.example.json
    │   ├── tags.example.json
    │   ├── naming-conventions.md
    ├── skill/
    │   ├── metadata_tagging/SKILL.md
    │   ├── source_to_note/SKILL.md
    │   ├── taxonomy_management/SKILL.md
    │   └── vault_health_check/SKILL.md
    └── AGENTS.md
```

### Folder Responsibilities

| Folder | Purpose | Rules |
|--------|---------|-------|
| `source/` | Human drops files here | No templates, no frontmatter, no manual tagging, no manual metadata. User only drops files. |
| `.digest/` | Temporary AI workspace | OCR output, extracted text, intermediate summaries. Not permanent. |
| `notes/` | Final knowledge repository | Flat structure, no subfolders. Search-first architecture. |
| `assets/` | Attachments storage | Images, diagrams. Managed by health check. |
| `archive/` | Cold storage for processed notes | Deprecated generated notes, retired content. Source files are not archived automatically. |

---

## Workflow: End-to-End Processing

When asked to process the vault or when new files appear in `source/`, run the agents in this order:

### Step 1: Metadata Tagging

1. Scan `source/` for files without a corresponding `filename.ext.metadata.json` sidecar.
2. **Read and understand the source material.** Determine what the content is about. Derive a clear 4-12 word descriptive name following the rename rules in `.system/templates/naming-conventions.md`. If the current filename is not already a good descriptive name within 4-12 words, rename it. This understanding also informs type classification.
3. Normalize source filenames to lowercase kebab-case. Do not modify file content.
4. For each new file, generate a metadata sidecar using `.system/templates/file.metadata.json`.
5. Classify the `type` based on file extension and content.
6. Assign an immutable `id` using the ID generation standard (see below).
7. Set `status` to `new` and `created_at` to current Unix epoch seconds.
8. Write `source/filename.ext.metadata.json`.

### Step 2: Note Generation

1. Find source files in `source/` with `status: new` in their metadata sidecar.
2. Update the metadata `status` from `new` to `digest`.
3. Read and analyze the source file content. If the source is a `chats`, `webclip`, or `links` type with conversational content between a human and an LLM, apply the chat processing rules in `.system/skill/source_to_note/SKILL.md`.
4. Determine knowledge density. Decide how many atomic notes to produce.
5. Generate one or more notes in `notes/` following `.system/templates/note.md`.
6. Derive each note filename from its title by slugifying to lowercase kebab-case.
7. Assign `categories` and `tags` using the taxonomy indexes. If indexes don't exist, create them following the example templates.
8. Verify filename-title parity for every generated note — ensure the `slug` field matches the note filename (without `.md`).
9. Set metadata `status` to `publish`.

### Step 3: Taxonomy Management

1. Ensure `notes/categories.index.json` and `notes/tags.index.json` exist. If not, create them following `.system/templates/categories.example.json` and `.system/templates/tags.example.json`.
2. Verify all categories and tags used by notes are present in the index files.
3. Add any missing values.
4. Check for and remove duplicate entries.
5. Normalize vocabulary — ensure consistent `lowercase_snake_case`.

### Step 4: Vault Health Check

Run the validations listed in the **Validation Criteria** section below. Produce a report at `.digest/vault-health-check-report.md`.

---

## Metadata Contract

### Sidecar Format

Every source file MUST have a sidecar metadata file. The metadata filename includes the original file extension:

```text
source/filename.ext
source/filename.ext.metadata.json
```

### Metadata Schema

Template: `.system/templates/file.metadata.json`

```json
{
  "schema_version": 1,
  "id": "xxxxxxxx",
  "created_at": "1717369260",
  "type": "other",
  "status": "new"
}
```

### Status Lifecycle

```text
new ──→ digest ──→ publish
```

- `new` — Freshly detected, ready for processing.
- `digest` — Content extraction and note generation in progress.
- `publish` — Notes generated and written to `notes/`.

There is no `failed` status. Files that cannot be processed reset to `new` for retry.

### Type Values

| Type | Description |
|------|-------------|
| `text` | Plain text, markdown, code files |
| `webpage` | Full web pages, HTML files |
| `pdf` | PDF documents |
| `image` | PNG, JPG, GIF, screenshots, photos |
| `json` | JSON data files |
| `links` | URL collections, link dumps |
| `documents` | DOC, DOCX, spreadsheets, presentations, manuals, datasheets |
| `webclip` | Obsidian webclipper content, saved snippets |
| `audio` | MP3, WAV, voice notes |
| `video` | MP4, video content |
| `chats` | Chat exports |
| `other` | Unclassified content |



### ID Generation Standard

All IDs are immutable 8-character hex strings (no prefix).

**Formula:**

```text
1. Normalize source filename to lowercase kebab-case
2. Record created_at as Unix epoch seconds
3. Hash input:  normalized-filename_epoch
   Example:     my-manual.pdf_1717369260
4. SHA-256 → first 8 hex chars
5. Result:     a7f3b2c1
```

**Note Inheritance:** All notes derived from a source inherit the source ID. A single source producing multiple notes → all notes share the same `id` field. The `id` itself traces provenance; no separate `derived_from` field is needed.

No vault lookup is required. No duplicate checking is required.

---

## Naming Conventions

All vault-managed filenames follow the complete rules in `.system/templates/naming-conventions.md`.

**Key rules (summary):**
- Lowercase kebab-case, hyphens between words, no spaces or underscores.
- 4-12 descriptive words. All words must carry meaning. 3 words or fewer always triggers a rename.
- Lead with the most dominant concept. Follow with other relevant concepts if space allows.
- Source and note filenames are independent — connected by the shared `id`, not by filename similarity.

---

## Note Generation Rules

### Atomic Note Rules

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

### Title Rules

Note titles must follow the rules in `.system/templates/naming-conventions.md`:
- 4-12 words, all meaningful.
- Lead with the most dominant concept.
- Run the meaningfulness self-check (banned patterns, ≥ 3 non-stop words, ≥ 50% meaningful ratio) on every title before writing.
- The title alone should inform what the note is about.

**Examples:**

```text
Good: how-transformer-attention-works-internally    (5 words, meaningful)
Bad:  transformer                                    (1 word, too vague)

Good: rclone-virtual-backends-explained             (4 words, passes 4-word minimum)
Bad:  rclone-virtual-backends                        (3 words, under minimum)

Good: container-orchestration-with-kubernetes       (4 words, 1 stop = 75% meaningful)
Bad:  a-comprehensive-guide-to-container-orchestration  (7 words, 3 stops = 57%, padded)
```

### Note Template

All generated notes must follow `.system/templates/note.md`:

```markdown
---
id: ""
slug: ""
date: ""
categories:
  -
tags:
  -
summary: ""
---

# {{title}}

## Summary

{{summary}}

---

## Core Concepts

- Concept 1
- Concept 2
- Concept 3

---

## Details

Main knowledge content.
```

---

## Taxonomy System

Categories and tags are controlled vocabularies managed through index files.

**Structure guides:** `.system/templates/categories.example.json` and `.system/templates/tags.example.json`

**Runtime indexes:** `notes/categories.index.json` and `notes/tags.index.json`

**Rules:**
- Taxonomy values use lowercase snake_case.
- Categories represent broad domains. Expected count: 20-50. New category creation should be rare.
- Tags represent specific concepts. Tag count may grow indefinitely.
- Always search existing values before creating new ones.
- Reuse existing values whenever possible.
- Append new values to the runtime indexes in `notes/`.
- Every entry must have a `name` and `description`.

---

## Agents

The vault has four specialized agents, each with a corresponding skill file:

### Metadata Agent

- Scan `source/` for new files.
- Read and understand source content. Derive a 4-12 word descriptive name following `.system/templates/naming-conventions.md`. Rename only if the current name is not already a good descriptive name.
- Normalize filenames to lowercase kebab-case.
- Generate `filename.ext.metadata.json` sidecars.
- Classify `type`.
- Assign immutable IDs (after rename and normalization).
- Never modify source file content.

**Skill file:** `.system/skill/metadata_tagging/SKILL.md`

### Note Generation Agent

- Read source material and metadata.
- Update metadata status to `digest`.
- Extract and analyze content.
- Determine knowledge density and decide note count.
- Generate notes following `.system/templates/note.md`.
- Derive filename from title (slugify to lowercase kebab-case).
- Assign categories and tags from taxonomy indexes.
- Generate summaries.
- Verify filename-title parity before marking `publish`.

**Skill file:** `.system/skill/source_to_note/SKILL.md`

### Taxonomy Agent

- Manage `notes/categories.index.json` and `notes/tags.index.json`.
- Search existing values before creating new ones.
- Prevent duplicate entries.
- Normalize vocabulary.

**Skill file:** `.system/skill/taxonomy_management/SKILL.md`

### Health Check Agent

- Detect orphan assets.
- Detect broken links.
- Detect missing metadata.
- Detect failed processing.
- Detect invalid categories and tags.
- Detect schema violations.
- Detect filename-title mismatches.
- Detect stale notes.
- Detect taxonomy drift.

**Skill file:** `.system/skill/vault_health_check/SKILL.md`

---

## Global Instructions

1. Never modify source file content. Source filenames may be normalized or renamed when organizing the vault.
2. Always preserve provenance.
3. Always generate atomic notes.
4. Prefer existing taxonomy values.
5. Create new taxonomy values only when required.
6. Knowledge notes must be reusable.
7. Avoid conversational writing.
8. Prefer factual statements.
9. Prefer permanent knowledge over temporary observations.
10. Every note must contain a summary.
11. Every note must have categories and tags.
12. Every note must have a unique ID.
13. The `slug` field in frontmatter must match the note filename (without `.md`).
14. Search and metadata are primary retrieval methods.
15. Folder hierarchy must not be used for organization.
16. Note titles must follow `.system/templates/naming-conventions.md`: 4-12 meaningful words, lead with dominant concept, pass meaningfulness self-check. The title alone should inform what the note is about.
17. Note filenames must match note titles. Derive the filename by slugifying the title to lowercase kebab-case and appending `.md`. Do not use the source file name as the note filename.

---

## Validation Criteria

Every processed note must pass these checks before the vault is considered healthy:

1. **Metadata Sidecar Exists** — Every source file has a `filename.ext.metadata.json` sidecar.
2. **Metadata Status is Publish** — The sidecar `status` is `publish`.
3. **Note Title Follows Naming Rules** — Title is 4-12 words (mechanical check). All words must be meaningful (generation rule, not health-checked).
4. **Slug Matches Filename** — The `slug` field in frontmatter matches the note filename (without `.md`).
5. **Categories Exist in Taxonomy** — All note categories are present in `categories.index.json`.
6. **Tags Exist in Taxonomy** — All note tags are present in `tags.index.json`.
7. **Source Content Not Modified** — The original source file was never edited.
8. **No Banned Patterns** — Title/source filename contains no hex strings, UUIDs, numeric-dominant words, or single-letter words (see `.system/templates/naming-conventions.md` Check A).
9. **Minimum Meaningful Words** — Title/source filename has ≥ 3 non-stop words (see naming-conventions.md Check B).
10. **Stop-Word Ratio** — Title/source filename has ≥ 50% meaningful words (see naming-conventions.md Check C).

These checks are enforced by the **Health Check Agent**. Additional checks:
- Stalled processing — metadata sidecars stuck at `status: digest`.
- Orphan assets — files in `assets/` not referenced by any note.
- Taxonomy drift — duplicate or near-duplicate category/tag entries.
- Schema violations — notes or metadata files that do not follow the defined schema.
- Stale notes — notes that have not been updated in a long time.

The health check produces a report at `.digest/vault-health-check-report.md`.

---

## Success Criteria

The system is considered successful when:

1. User only drops files into `source/`.
2. Metadata is generated automatically.
3. AI converts source material into atomic knowledge notes.
4. Categories and tags remain controlled.
5. Every note is traceable to its source.
6. Generated notes pass filename-title parity checks.
7. Notes inherit the source ID and provenance is traceable.
8. Health check finds no schema, taxonomy, metadata, or filename-title violations.
9. Search remains effective at large scale.
10. Vault scales beyond 100,000 notes without structural redesign.
