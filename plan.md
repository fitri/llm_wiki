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

---

## High Level Architecture

```text
Human
│
▼
raw/
│
▼
metadata_tagging skill
│
▼
filename.ext.metadata.json
│
▼
processing/
│
▼
raw_to_note skill
│
▼
notes/
│
│
├── taxonomy_management  (controlled vocabularies)
│
└── vault_health_check   (validates metadata, taxonomy, filename-title, etc.)
│
▼
search + metadata + AI retrieval
```

---

## Vault Structure

```text
vault/
│
├── raw/
├── processing/
├── notes/
├── assets/
├── archive/
│
└── schema/
    │
    ├── templates/
    │   ├── note.md
    │   └── file.metadata.json
    │
    ├── taxonomy/
    │   ├── categories.index.json
    │   ├── tags.index.json
    │   └── aliases.index.json
    │
    ├── skill/
    │   ├── metadata_tagging/
    │   │   └── SKILL.md
    │   │
    │   ├── raw_to_note/
    │   │   └── SKILL.md
    │   │
    │   ├── taxonomy_management/
    │   │   └── SKILL.md
    │   │
    │   └── vault_health_check/
    │       └── SKILL.md
    │
    ├── agents.md
    └── instructions.md
```

---

## Folder Responsibilities

### raw/

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
Drop file into raw/
```

Only.

---

### processing/

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

Raw files are not archived automatically.

---

## Metadata Contract

Every raw file MUST have a sidecar metadata file that includes the original file extension in the metadata filename.

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

File: `schema/templates/file.metadata.json`

```json
{
  "schema_version": 1,
  "id": "raw_xxxxxxxx",
  "created_at": "",
  "status": "detected",
  "content_type": "other",
  "source_type": "other",
  "processing_attempts": 0,
  "last_processed_at": null,
  "generated_notes": [],
  "error": null
}
```

---

### Metadata Status Values

Allowed values:

```text
detected
processing
processed
failed
archived
```

---

### Confidence Values

Allowed values:

```text
high
medium
low
```

Definitions:

1. `high` means the note is strongly supported by the source material.
2. `medium` means the note is mostly supported but has some uncertainty.
3. `low` means the note is useful but requires human review.

---

### Content Type Enumeration

Allowed values:

```text
text
pdf
image
audio
video
url
chat_export
email
other
```

No additional values permitted without updating schema.

---

### Source Type Enumeration

Examples:

```text
youtube_link
website_link
manual
datasheet
meeting_note
voice_note
chatgpt_export
email_export
screenshot
photograph
other
```

May expand over time.

---

### ID Generation Standard

All IDs are immutable.

**Format:**

```text
raw_<hash>
nt_<hash>
```

**Generation:**

```text
timestamp
↓
hash
↓
id
```

**Examples:**

```text
raw_a1b2c3d4
nt_f6g7h8i9
```

No vault lookup required.

No duplicate checking required.

---

### Filename Naming Convention

All vault-managed filenames SHOULD use lowercase kebab-case.

Words separated by hyphens.

Examples:

```text
raw/my-downloaded-manual.pdf
raw/youtube-video-about-rag.txt
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
6. Generated note filenames must match the note title. Derive the filename by slugifying the title to lowercase kebab-case. Do not use the raw source name as the filename.
7. Raw file content must not be edited, but raw filenames may be normalized or renamed by AI when organizing the vault.

---

### Knowledge Note Template

File: `schema/templates/note.md`

```markdown
---
schema_version: 1

id: nt_xxxxxxxx

title: ""

created_at: ""
updated_at: ""

status: processed

confidence: high

categories:
  -

tags:
  -

summary: ""

derived_from:
  - raw_xxxxxxxx

related_notes:
  -

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

---

## Related Notes

- [[Related Note]]

---

## Sources

- raw_xxxxxxxx

---

## AI Notes

Generated from source material.
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
5. Raw summaries that do not introduce reusable knowledge.
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
schema/templates/note.md
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

Taxonomy values use lowercase snake_case.

**Files:**

```text
schema/taxonomy/categories.index.json
schema/taxonomy/tags.index.json
schema/taxonomy/aliases.index.json
```

---

### Category Rules

Categories represent broad domains.

**Examples:**

```text
ai
semiconductor
software_engineering
homelab
career
leadership
personal
finance
science
```

**Expected count:** 20-50 categories

AI should prefer existing categories. New category creation should be rare.

---

### Tag Rules

Tags represent concepts.

**Examples:**

```text
riscv
docker
pytest
transformer
attention
rag
vector_database
```

Tag count may grow indefinitely.

AI should:

1. Search existing tags.
2. Reuse if possible.
3. Create if necessary.
4. Update `tags.index.json`.

---

### Alias Rules

**Purpose:** Prevent vocabulary drift.

**Example:**

```json
{
  "llm": "large_language_model",
  "ml": "machine_learning"
}
```

---

## Agent Definitions

File: `schema/agents.md`

### Metadata Agent

**Responsibilities:**

- Scan `raw/`
- Detect new files
- Generate metadata
- Classify `content_type`
- Classify `source_type`

**Output:** `filename.ext.metadata.json`

---

### Note Generation Agent

**Responsibilities:**

- Read raw material
- Read metadata
- Extract content
- Determine knowledge density
- Decide note count
- Generate notes
- Assign categories
- Assign tags
- Generate summaries
- Create links
- Derive note filename from title (slugify to lowercase kebab-case)
- Verify filename-title parity before marking processed
- Update `generated_notes`

**Output:** `notes/*.md`

---

### Taxonomy Agent

**Responsibilities:**

- Manage categories index
- Manage tags index
- Manage aliases
- Prevent duplicates
- Normalize vocabulary

---

### Health Check Agent

**Responsibilities:**

- Detect orphan assets
- Detect broken links
- Detect missing metadata
- Detect failed processing
- Detect invalid categories
- Detect invalid tags
- Detect schema violations
- Detect filename-title mismatches
- Detect stale notes
- Detect taxonomy drift

---

## Skill Definitions

### metadata_tagging/SKILL.md

**Purpose:** Generate metadata sidecar files.

**Input:** `raw/*`

**Output:** `filename.ext.metadata.json`

**Rules:**

- Never modify original file content.
- Generate metadata only.
- Classify content.
- Assign ID.
- Initialize processing state.
- Use lowercase kebab-case for generated filenames.

---

### raw_to_note/SKILL.md

**Purpose:** Convert raw material into atomic knowledge notes.

**Responsibilities:**

- Analyze source
- Extract content
- Determine note count
- Generate notes
- Assign metadata
- Link related notes
- Update provenance

**Filename rules:**

1. Generated notes must use lowercase kebab-case filenames.
2. Notes must use `.md` extension.
3. Note filenames must match note titles. Derive the filename by slugifying the title to lowercase kebab-case. Do not use the raw source name as the filename.

**Guiding principle:**

```text
Create knowledge, not summaries.
```

---

### taxonomy_management/SKILL.md

**Purpose:** Manage controlled vocabulary.

**Responsibilities:**

- Reuse existing categories
- Reuse existing tags
- Create new values only when required
- Update taxonomy indexes

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

**Filename rules:**

1. When organizing or renaming files during health check, use lowercase kebab-case.
2. Attachments should use lowercase kebab-case when organized by AI.
3. Raw file content must not be modified.
4. Raw filenames may be normalized or renamed by AI when organizing the vault.

---

## Global Instructions

File: `instructions.md`

**Rules:**

1. Never modify raw file content. Raw filenames may be normalized or renamed when organizing the vault.
2. Always preserve provenance.
3. Always generate atomic notes.
4. Prefer existing taxonomy values.
5. Create new taxonomy values only when required.
6. Knowledge notes must be reusable.
7. Avoid conversational writing.
8. Prefer factual statements.
9. Prefer permanent knowledge over temporary observations.
10. Every note must contain a summary.
11. Every note must contain source references.
12. Every note must have categories and tags.
13. Every note must have a unique ID.
14. Search and metadata are primary retrieval methods.
15. Folder hierarchy must not be used for organization.
16. Note titles must be descriptive. Avoid single-word titles unless the word is a recognized proper name. Less than 18 words; aim for fewer, not more. The title alone should inform what the note is about.
17. Note filenames must match note titles. Derive the filename by slugifying the title to lowercase kebab-case and appending `.md`. Do not use the raw source name as the filename.

---

## Validation & Health Checks

Every processed note must pass these automated checks before the vault is considered healthy:

1. **Metadata Sidecar Exists** — Every raw file has a `filename.ext.metadata.json` sidecar.
2. **Metadata Status is Processed** — The sidecar status is `processed`, not `detected` or `failed`.
3. **Generated Notes Populated** — The `generated_notes` array in the metadata contains at least one note ID.
4. **Note ID Matches Metadata** — Each note's frontmatter `id` exists in its source metadata `generated_notes`.
5. **Note Title is Descriptive** — Title avoids single-word (unless proper name), is less than 18 words.
6. **Filename Matches Title** — The note filename equals the slugified title: lowercase kebab-case + `.md`.
7. **Categories Exist in Taxonomy** — All note categories are present in `categories.index.json`.
8. **Tags Exist in Taxonomy** — All note tags are present in `tags.index.json`.
9. **Raw Content Not Modified** — The original raw file was never edited.
10. **Wikilinks Resolve** — All `[[wikilinks]]` point to existing notes.

These checks are enforced by the **Vault Health Check Agent** and may be run on demand or as part of a CI/CD pipeline.

---

## Success Criteria

The system is considered successful when:

1. User only drops files into `raw/`.
2. Metadata is generated automatically.
3. AI converts raw material into atomic knowledge notes.
4. Categories and tags remain controlled.
5. Every note is traceable to its source.
6. Generated notes pass filename-title parity checks.
7. Processed metadata points to generated note IDs.
8. Health check finds no schema, taxonomy, metadata, or filename-title violations.
9. Search remains effective at large scale.
10. Vault scales beyond 100,000 notes without structural redesign.
