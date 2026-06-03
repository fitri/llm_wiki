# Obsidian Knowledge Warehouse - Master Implementation Plan (V1)

## Goal

Build an AI-assisted knowledge warehouse on top of Obsidian.

### Design Principles

1. Human effort must be minimized.
2. Information capture must be frictionless.
3. Metadata drives organization.
4. Search drives retrieval.
5. AI performs classification and note generation.
6. Knowledge must be traceable to original sources.
7. The system must scale to 100,000+ notes.
8. The system must work across multiple AI models.
9. The filesystem is storage, not organization.
10. Categories and tags are controlled vocabularies.
11. PLAN.md is the project build guide. No operational file (skills, templates, AGENTS.md, notes) may reference or depend on it.

---

## High Level Architecture

```text
Human
│
▼
source/
│
▼
metadata_tagging skill
│
▼
filename.ext.metadata.json
│
▼
source_to_note skill
│
▼
notes/
│
▼
taxonomy_management skill (controlled vocabularies)
│
▼
vault_health_check skill (validates metadata, taxonomy, etc.)
│
▼
search + metadata + AI retrieval

.digest/ — temporary AI workspace (OCR, extracted text, intermediate artifacts)
```

## Usage / Invocation

After dropping a file into `source/`, invoke the pipeline by running:

```text
opencode run "digest"
```

OpenCode reads `AGENTS.md` (the operating guide) and executes the four-stage
pipeline: metadata tagging → note generation → taxonomy management → vault
health check. Any prompt indicating agent activation (e.g. "digest", "run",
"process the vault") will trigger the same flow.

---

## Vault Structure

```text
vault/
│
├── source/
├── .digest/
├── notes/
├── assets/
├── archive/
│
├── AGENTS.md (symlink → .system/AGENTS.md)
│
└── .system/
    │
    ├── templates/
    │   ├── note.md
    │   ├── file.metadata.json
    │   ├── categories.example.json
    │   └── tags.example.json
    │
    ├── skill/
    │   ├── metadata_tagging/
    │   │   └── SKILL.md
    │   │
    │   ├── source_to_note/
    │   │   └── SKILL.md
    │   │
    │   ├── taxonomy_management/
    │   │   └── SKILL.md
    │   │
    │   └── vault_health_check/
    │       └── SKILL.md
    │
    └── AGENTS.md
```

---

## Folder Responsibilities

### source/

**Purpose:** Human ingestion zone.

Users may drop:

- pdf
- txt
- md
- png
- jpg
- mp3
- wav
- urls (one URL per line in a `.txt` or `.url` file)
- chat exports
- emails
- any supported file

**Rules:**

- No templates
- No frontmatter
- No categorization
- No manual tagging
- No manual metadata
- No manual renaming

**User action:**

```text
Drop file into source/
```

Only.

---

### .digest/

**Purpose:** Temporary AI workspace.

Contains:

- OCR output
- extracted text
- intermediate summaries
- processing artifacts

No permanent knowledge stored here.

---

### notes/

**Purpose:** Final knowledge repository.

**Rules:**

- Flat structure
- No subfolders
- Search-first architecture
- Metadata-driven retrieval

---

### assets/

**Purpose:** Storage for:

- images
- diagrams
- attachments

Managed through health-check automation.

---

### archive/

**Purpose:** Cold storage for processed notes only.

Contains:

- deprecated generated notes
- retired processed content

Source files are not archived automatically.

---

## Metadata Contract

Every source file MUST have a sidecar metadata file that includes the original file extension in the metadata filename.

```text
filename.ext
filename.ext.metadata.json
```

Example:

```text
manual.pdf
manual.pdf.metadata.json
```

---

### Metadata Template

File: `.system/templates/file.metadata.json`

```json
{
  "schema_version": 1,
  "id": "xxxxxxxx",
  "created_at": "1717369260",
  "type": "other",
  "status": "new"
}
```

---

### Metadata Status Values

Allowed values:

```text
new     Freshly detected, ready for processing
digest  Content extraction and note generation in progress
publish Notes generated and written to notes/
```

Lifecycle:

```text
new ──→ digest ──→ publish
```

No `failed` status. Files that cannot be processed reset to `new` for retry.

---

### Type Values

Single field replacing both `content_type` and `source_type`.

Allowed values:

```text
text      Plain text, markdown, code files
webpage   Full web pages, HTML files
pdf       PDF documents
image     PNG, JPG, GIF, screenshots, photos
json      JSON data files
links     URL collections, link dumps
documents DOC, DOCX, spreadsheets, presentations, manuals, datasheets
webclip   Obsidian webclipper content, saved snippets
audio     MP3, WAV, voice notes
video     MP4, video content
chats     Chat exports
other     Unclassified content
```

---

<!-- Source Type Enumeration removed — merged into single Type Values field above -->

---

### ID Generation Standard

All IDs are immutable.

**Format:**

```text
xxxxxxxx  (8-char hex hash, no prefix)
```

**Generation:**

```text
1. Normalize source filename to lowercase kebab-case
2. Record created_at as Unix epoch seconds
3. Hash input:  normalized-filename_epoch
   Example:     my-manual.pdf_1717369260
4. SHA-256 → first 8 hex chars
5. Result:     a7f3b2c1
```

**Note Inheritance:**

All notes derived from a source inherit the source ID. A single source producing multiple notes → all notes share the same `id` field. The `id` itself traces provenance; `derived_from` is not needed.

No vault lookup required. No duplicate checking required.

---

### Filename Naming Convention

All vault-managed filenames SHOULD use lowercase kebab-case.

Words separated by hyphens.

Examples:

```text
source/my-downloaded-manual.pdf
source/youtube-video-about-rag.txt
notes/how-transformer-attention-works.md
assets/transformer-attention-diagram.png
archive/old-semiconductor-reference.md
```

Rules:

1. Use lowercase letters.
2. Use `-` between words.
3. Do not use spaces.
4. Do not use underscores in filenames.
5. Do not use special characters unless required by the file extension.
6. Generated note filenames must match the note title. Derive the filename by slugifying the title to lowercase kebab-case. Do not use the source file name as the note filename.
7. Source file content must not be edited, but source filenames may be normalized or renamed by AI when organizing the vault.

---

### Knowledge Note Template

File: `.system/templates/note.md`

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

### Atomic Note Rules

A knowledge note should contain one reusable idea.

Create a separate note when the source contains:

1. A distinct concept.
2. A distinct process or workflow.
3. A distinct decision or conclusion.
4. A distinct rule, principle, or best practice.
5. A distinct technical reference.
6. A distinct troubleshooting pattern.
7. A distinct comparison between ideas, tools, or methods.
8. A distinct reusable question-and-answer pair.

Do not create separate notes for:

1. Minor details.
2. Repeated statements.
3. Temporary observations.
4. Personal commentary with no reusable value.
5. Source summaries that do not introduce reusable knowledge.
6. Content that only makes sense inside the original document.

Each note must be understandable without reading the full original source.

Each note should answer at least one of these questions:

1. What is this?
2. How does this work?
3. Why does this matter?
4. When should this be used?
5. What problem does this solve?
6. What should be remembered from this?

If a source contains many independent concepts, generate multiple notes.

If a source contains one tightly connected topic, generate one note.

Prefer fewer high-quality reusable notes over many shallow notes.

Do not force-split a single source into multiple notes. Only split when topics are truly too big and diverse to fit in one note. A single project, tool, or article homepage should typically produce one note.

### Title Naming Rule

Note titles must be descriptive. Avoid single-word titles unless the word is a recognized proper name (e.g. "Rclone", "LiteLLM", "9Router"). Keep titles concise: less than 18 words, but aim for fewer — minimize, not maximize. The title alone should inform what the note is about.

Examples:

```text
Good: How Transformer Attention Works    (4 words)
Bad:  Transformer                         (too vague, single word)

Good: Rclone Virtual Backends            (3 words, descriptive)
Bad:  Virtual Backends                    (doesn't name the tool)

Good: Container Orchestration with Kubernetes  (4 words)
Bad:  A Comprehensive Guide to Container Orchestration Using Kubernetes  (10 words, padded)
```

---

### Generated Note Rule

All generated notes must follow:

```text
.system/templates/note.md
```

Generated notes must be standard markdown files.

The generated note filename must use lowercase kebab-case and must match the note title. Derive the filename by slugifying the title to lowercase kebab-case.

Example:

```text
Title: How Transformer Attention Works
Filename: how-transformer-attention-works.md
```

---

## Taxonomy System

Categories and tags are controlled vocabularies.

Runtime indexes live in `notes/categories.index.json` and `notes/tags.index.json`. Structure guides are at `.system/templates/categories.example.json` and `.system/templates/tags.example.json`.

Taxonomy values use lowercase snake_case.

---

### Category Rules

Categories represent broad domains.

**Expected count:** 20-50 categories

AI should:
1. Read `.system/templates/categories.example.json` for structure guidance.
2. Read `notes/categories.index.json` for existing values. Create if missing.
3. Reuse existing values. New category creation should be rare.

---

### Tag Rules

Tags represent concepts.

Tag count may grow indefinitely.

AI should:

1. Read `.system/templates/tags.example.json` for structure guidance.
2. Read `notes/tags.index.json` for existing values. Create if missing.
3. Search existing tags. Reuse if possible. Create if necessary.
4. Append new tags to `notes/tags.index.json`.

---

## Agent Definitions

File: `.system/AGENTS.md`

### Metadata Agent

- Scan `source/` for new files.
- Read and understand source content to evaluate filenames and classify types.
- Rename source files to descriptive 5–15 word lowercase kebab-case names when the current name is vague or underspecified.
- Generate `filename.ext.metadata.json` sidecars.
- Classify `type`.
- Assign immutable IDs.
- Never modify source file content.

**Skill file:** `.system/skill/metadata_tagging/SKILL.md`

---

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

---

### Taxonomy Agent

- Manage `notes/categories.index.json` and `notes/tags.index.json`.
- Search existing values before creating new ones.
- Prevent duplicate entries.
- Normalize vocabulary.

**Skill file:** `.system/skill/taxonomy_management/SKILL.md`

---

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

## Skill Definitions

### metadata_tagging/SKILL.md

**Purpose:** Generate metadata sidecar files.

**Input:** `source/*`

**Output:** `filename.ext.metadata.json`

**Rules:**

- Never modify original file content.
- Read and understand source content before renaming or classifying.
- Rename files to descriptive 5–15 word lowercase kebab-case names when the original name is vague or underspecified.
- Generate metadata only.
- Classify content.
- Assign ID.
- Initialize processing state.
- Use lowercase kebab-case for generated filenames.

---

### source_to_note/SKILL.md

**Purpose:** Convert source material into atomic knowledge notes.

**Responsibilities:**

- Analyze source
- Extract content
- Determine note count
- Generate notes
- Assign metadata
- Update provenance

**Filename rules:**

1. Generated notes must use lowercase kebab-case filenames.
2. Notes must use `.md` extension.
3. Note filenames must match note titles. Derive the filename by slugifying the title to lowercase kebab-case. Do not use the source file name as the note filename.

**Guiding principle:**

```text
Create knowledge, not summaries.
```

---

### taxonomy_management/SKILL.md

**Purpose:** Manage controlled vocabulary for categories and tags.

**Responsibilities:**

- Read `.system/templates/*.example.json` for structure guidance
- Read `notes/categories.index.json` and `notes/tags.index.json` for existing values
- Reuse existing categories and tags
- Create new values only when required
- Append new values to runtime indexes in `notes/`

---

### vault_health_check/SKILL.md

**Purpose:** Maintain vault integrity.

**Checks:**

- broken links
- missing metadata
- orphan assets
- failed processing
- invalid categories
- invalid tags
- schema violations
- filename-title mismatches
- stale notes
- taxonomy drift

**Filename rules:**

1. When organizing or renaming files during health check, use lowercase kebab-case.
2. Attachments should use lowercase kebab-case when organized by AI.
3. Source file content must not be modified.
4. Source filenames may be normalized or renamed by AI when organizing the vault.

---

## Global Instructions

File: `.system/AGENTS.md`

**Rules:**

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

## Validation & Health Checks

Every processed note must pass these automated checks before the vault is considered healthy:

1. **Metadata Sidecar Exists** — Every source file has a `filename.ext.metadata.json` sidecar.
2. **Metadata Status is Publish** — The sidecar status is `publish`.
3. **Note Title is Descriptive** — Title avoids single-word (unless proper name), is less than 18 words.
4. **Slug Matches Filename** — The `slug` field in frontmatter matches the note filename (without `.md`).
5. **Categories Exist in Taxonomy** — All note categories are present in `categories.index.json`.
6. **Tags Exist in Taxonomy** — All note tags are present in `tags.index.json`.
7. **Source Content Not Modified** — The original source file was never edited.

These checks are enforced by the **Vault Health Check Agent** and may be run on demand or as part of a CI/CD pipeline.

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
