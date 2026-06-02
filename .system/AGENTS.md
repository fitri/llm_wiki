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
    │   └── tags.example.json
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
2. **Read and understand the source material.** Determine what the content is about. If the current filename does not accurately describe the content — or is too short, vague, or underspecified (e.g. single words, generic names) — rename it to a descriptive 5–15 word lowercase kebab-case name that reflects the content. This understanding also informs type classification.
3. Normalize source filenames to lowercase kebab-case. Do not modify file content.
4. For each new file, generate a metadata sidecar using `.system/templates/file.metadata.json`.
5. Classify the `type` based on file extension and content.
6. Assign an immutable `id` using the ID generation standard (see below).
7. Set `status` to `new` and `created_at` to current Unix epoch seconds.
8. Write `source/filename.ext.metadata.json`.

### Step 2: Note Generation

1. Find source files in `source/` with `status: new` in their metadata sidecar.
2. Update the metadata `status` from `new` to `digest`.
3. Read and analyze the source file content.
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

All vault-managed filenames use lowercase kebab-case. Words are separated by hyphens.

**Rules:**

1. Use lowercase letters.
2. Use `-` between words.
3. Do not use spaces.
4. Do not use underscores in filenames.
5. Do not use special characters unless required by the file extension.
6. Generated note filenames must match the note title. Derive the filename by slugifying the title to lowercase kebab-case. Do not use the source file name as the note filename.
7. Source file content must not be edited, but source filenames may be normalized or renamed by AI when organizing the vault.

**Examples:**

```text
source/my-downloaded-manual.pdf
notes/how-transformer-attention-works.md
assets/transformer-attention-diagram.png
archive/old-semiconductor-reference.md
```

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

Note titles must be descriptive. Avoid single-word titles unless the word is a recognized proper name (e.g. "Rclone", "LiteLLM"). Keep titles concise: less than 18 words, but aim for fewer. The title alone should inform what the note is about.

**Examples:**

```text
Good: How Transformer Attention Works    (4 words, descriptive)
Bad:  Transformer                         (too vague, single word)

Good: Rclone Virtual Backends            (3 words, descriptive)
Bad:  Virtual Backends                    (doesn't name the tool)

Good: Container Orchestration with Kubernetes  (4 words)
Bad:  A Comprehensive Guide to Container Orchestration Using Kubernetes  (10 words, padded)
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
- Read and understand source content to evaluate filenames and classify types.
- Rename source files to descriptive 5–15 word lowercase kebab-case names when the current name is vague or underspecified.
- Generate `filename.ext.metadata.json` sidecars.
- Classify `type`.
- Assign immutable IDs.
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
- Generate summaries and link related notes.
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
16. Note titles must be descriptive. Avoid single-word titles unless the word is a recognized proper name. Less than 18 words; aim for fewer, not more. The title alone should inform what the note is about.
17. Note filenames must match note titles. Derive the filename by slugifying the title to lowercase kebab-case and appending `.md`. Do not use the source file name as the note filename.

---

## Validation Criteria

Every processed note must pass these checks before the vault is considered healthy:

1. **Metadata Sidecar Exists** — Every source file has a `filename.ext.metadata.json` sidecar.
2. **Metadata Status is Publish** — The sidecar `status` is `publish`.
3. **Note Title is Descriptive** — Title avoids single-word (unless proper name), is less than 18 words.
4. **Slug Matches Filename** — The `slug` field in frontmatter matches the note filename (without `.md`).
5. **Categories Exist in Taxonomy** — All note categories are present in `categories.index.json`.
6. **Tags Exist in Taxonomy** — All note tags are present in `tags.index.json`.
7. **Source Content Not Modified** — The original source file was never edited.

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
