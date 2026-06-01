# Master Instructions

This file is the top-level operating guide for all AI agents managing this wiki vault. It is the first file every agent must read before performing any task within the vault.

**Role:** Operational guide for agent behavior, validation, and process flow.
**Companion:** `plan.md` is the master implementation plan — read it when the task involves architecture, rules, schema design, workflows, or success criteria changes.

---

## When You Are Invoked

If you receive this file alone, or a single-word instruction (e.g. "go", "run", "digest", "start", "execute"), your default task is to act as the Master Orchestrator and execute the full Standard Processing Flow end-to-end.

Map of common terse commands:
- "go", "run", "start", "execute", "digest" → Run full processing pipeline
- "health", "check" → Run Health Check Agent only
- "scan", "detect" → Run Metadata Agent only

When invoked as the Master Orchestrator, you perform ALL agent roles (Metadata → Note Generation → Taxonomy → Health Check) in sequence. Read schema files as needed per the Required Reading Order.

In all cases, do not ask for clarification — proceed immediately.

---

## Source Of Truth

These files define the vault's rules, workflows, and output shapes. Agents must treat them as authoritative.

| File | Purpose |
|---|---|
| `plan.md` | Master implementation plan — architecture, rules, workflows, success criteria |
| `instructions.md` | This file — operational master guide for all agents |
| `schema/agents.md` | Agent definitions and role responsibilities |
| `schema/skill/metadata_tagging/SKILL.md` | Metadata sidecar generation workflow |
| `schema/skill/raw_to_note/SKILL.md` | Note generation workflow |
| `schema/skill/taxonomy_management/SKILL.md` | Controlled vocabulary management |
| `schema/skill/vault_health_check/SKILL.md` | Vault integrity validation |
| `schema/templates/note.md` | Required shape for all generated knowledge notes |
| `schema/templates/file.metadata.json` | Required shape for all metadata sidecars |
| `schema/taxonomy/categories.index.json` | Approved category values |
| `schema/taxonomy/tags.index.json` | Approved tag values |
| `schema/taxonomy/aliases.index.json` | Vocabulary normalization mappings |

---

## Required Reading Order

For any task, agents must read files in this order:

1. **`instructions.md`** — Always first. This file.
2. **`plan.md`** — When the task touches architecture, rules, schema design, workflows, validation, or success criteria.
3. **`schema/agents.md`** — To understand which agent owns which responsibility.
4. **Relevant skill file** — `schema/skill/<skill>/SKILL.md` for the workflow being executed.
5. **Relevant templates** — `schema/templates/*` to produce correctly shaped output.
6. **Taxonomy indexes** — `schema/taxonomy/*.json` before assigning categories or tags to any note.

---

## Core Principles

1. **Raw content is immutable.** Never edit raw file content. Only filename normalization is permitted.
2. **Metadata and search drive organization.** Folders are storage only. Everything is found through metadata and search.
3. **Notes are flat.** All notes live in `notes/` with no subfolders.
4. **Provenance is required.** Every note must trace back to its source raw file.
5. **Controlled vocabularies must be respected.** Categories and tags come from taxonomy indexes only.
6. **Prefer small, correct changes.** Make minimal edits. Do not rewrite files unnecessarily.
7. **Validation is part of processing.** Do not mark work complete without verifying output. Health checks are not optional.
8. **Schema and plan must stay synchronized.** When a skill or rule changes in a SKILL.md or template file, update `plan.md`.

---

## Global Rules

These rules apply to all agents, all skills, all notes, and all metadata. No exceptions.

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

## Vault Directories

| Directory | Purpose | Rules |
|---|---|---|
| `raw/` | Human ingestion zone | Raw content immutable. Filenames may be normalized to lowercase kebab-case. No templates, no frontmatter, no manual tagging. |
| `processing/` | Temporary AI workspace | OCR output, extracted text, intermediate artifacts. No permanent knowledge stored here. |
| `notes/` | Final knowledge repository | Flat structure. No subfolders. Search-first architecture. All notes follow `schema/templates/note.md`. |
| `assets/` | Images, diagrams, attachments | Managed through health-check automation. |
| `archive/` | Cold storage for processed notes | Deprecated generated notes only. Raw files are not archived automatically. |
| `schema/` | Schema, skills, templates, taxonomy | The authoritative definition of how the vault operates. |

---

## Standard Processing Flow

```
1. User drops file into raw/

2. Metadata Agent scans raw/, normalizes raw filename,
   generates filename.ext.metadata.json, sets status: detected

3. Note Generation Agent reads raw file and metadata,
   sets status: processing,
   extracts content, decides note count,
   generates note(s) following schema/templates/note.md,
   derives filename from title (slugify to lowercase kebab-case),
   assigns categories and tags from taxonomy indexes,
   updates metadata generated_notes with note IDs,
   verifies filename-title parity,
   sets status: processed

4. Taxonomy Agent validates categories and tags against indexes,
   adds new values only when necessary

5. Health Check Agent validates output:
   metadata sidecar exists, status is processed,
   generated_notes populated, note IDs match,
   title is descriptive, filename matches title,
   categories and tags exist in taxonomy,
   wikilinks resolve, raw content was not modified
```

---

## Metadata Rules

- Every raw file MUST have a sidecar: `filename.ext.metadata.json`.
- The metadata filename includes the original file extension.
- Metadata status lifecycle: `detected → processing → processed`.
- Use `status: failed` only when processing cannot complete. Include error details.
- After successful processing, `generated_notes` must contain the generated note ID(s).
- Metadata IDs are immutable. Format: `raw_<hash>`.
- Raw filenames must be normalized to lowercase kebab-case before metadata generation.
- Do not process files that already have a metadata sidecar unless the status is `failed`.

### Metadata Template

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

## Note Generation Rules

- All generated notes must follow `schema/templates/note.md`.
- One reusable idea per note.
- Each note must be understandable without reading the full source.
- Prefer fewer high-quality reusable notes over many shallow notes.
- Do not force-split. A single source should typically produce one note unless topics are truly too diverse.
- A single project, tool, or article homepage should typically produce one note.
- Create knowledge, not summaries.
- Every note must answer at least one of: What is this? How does it work? Why does it matter? When should it be used? What problem does it solve? What should be remembered?

### Note Template

```markdown
---
schema_version: 1

id: nt_xxxxxxxx

title: "" # Descriptive, avoid single-word unless proper name, less than 18 words.

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

## Title And Filename Rules

### Title Rules

- Titles must be descriptive. Avoid single-word titles unless the word is a recognized proper name (e.g. "Rclone", "LiteLLM", "9Router").
- Less than 18 words; aim for fewer — minimize, not maximize.
- The title alone should inform what the note is about.

**Examples:**
```
Good: How Transformer Attention Works    (4 words)
Bad:  Transformer                         (too vague, single word)

Good: Rclone Virtual Backends            (3 words, descriptive)
Bad:  Virtual Backends                    (doesn't name the tool)

Good: LiteLLM AI Gateway                 (3 words, descriptive)
Bad:  LiteLLM                             (single word, lacks context)

Good: Container Orchestration with Kubernetes  (4 words)
Bad:  A Comprehensive Guide to Container Orchestration Using Kubernetes  (10 words, padded)
```

### Filename Rules

- Generated notes must use lowercase kebab-case filenames.
- Notes must use `.md` extension.
- Note filenames must match note titles. Derive the filename by slugifying the title to lowercase kebab-case.
- Do not use the raw source name as the filename.
- Verify filename-title parity before marking metadata `processed`.

**Example:**
```
Title:  How Transformer Attention Works
File:   how-transformer-attention-works.md

Title:  Rclone Cloud Storage Tool
File:   rclone-cloud-storage-tool.md

Title:  LiteLLM AI Gateway
File:   litellm-ai-gateway.md
```

---

## Taxonomy Rules

- Categories and tags are controlled vocabularies stored in `schema/taxonomy/`.
- Taxonomy values use lowercase snake_case.
- Categories represent broad domains. Expected count: 20-50. New category creation should be rare.
- Tags represent specific concepts. Count may grow indefinitely. New tags are allowed when needed.
- Aliases map common terms to canonical values to prevent vocabulary drift.
- Always search existing taxonomy values before creating new ones.
- Every category and tag entry must have a `name` and `description`.

### Required Read Before Assignment

Before assigning categories or tags to any note:
1. Read `schema/taxonomy/categories.index.json`.
2. Read `schema/taxonomy/tags.index.json`.
3. Reuse existing values whenever possible.
4. Create new values only when no existing value captures the concept.
5. Update the appropriate index file when adding new values.

---

## Agent Responsibilities

### Metadata Agent

- Scan `raw/` for new files.
- Normalize raw filenames to lowercase kebab-case.
- Generate `filename.ext.metadata.json` sidecars.
- Classify `content_type` and `source_type`.
- Assign immutable IDs.
- Never modify raw file content.

**Skill file:** `schema/skill/metadata_tagging/SKILL.md`

### Note Generation Agent

- Read raw material and metadata.
- Update metadata status to `processing`.
- Extract and analyze content.
- Determine knowledge density and decide note count.
- Generate notes following `schema/templates/note.md`.
- Derive filename from title (slugify to lowercase kebab-case).
- Assign categories and tags from taxonomy indexes.
- Generate summaries and link related notes.
- Update `generated_notes` in metadata.
- Verify filename-title parity before marking `processed`.

**Skill file:** `schema/skill/raw_to_note/SKILL.md`

### Taxonomy Agent

- Manage `categories.index.json`, `tags.index.json`, `aliases.index.json`.
- Search existing values before creating new ones.
- Prevent duplicate entries.
- Normalize vocabulary.

**Skill file:** `schema/skill/taxonomy_management/SKILL.md`

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

**Skill file:** `schema/skill/vault_health_check/SKILL.md`

---

## Health Check Requirements

Every processed note must pass these checks before the vault is considered healthy:

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

Report output: `processing/vault-health-check-report.md`.

---

## When Updating Schema

Follow this protocol when changing any schema file:

1. **Update `plan.md` first** — When changing architecture, rules, validation requirements, or workflows.
2. **Update the specific SKILL.md** — When changing workflow behavior or rules for a specific skill.
3. **Update template files** — When changing required output structure for notes or metadata.
4. **Update this file** — When rules, validation checks, processing flow, or agent responsibilities change.
5. **Keep files synchronized** — `instructions.md`, `plan.md`, and skill files must not contradict each other.
6. **Prefer `must` over `should`** — Use `should` for guidance, `must` only for hard requirements you intend to enforce.

---

## Completion Checklist

Before reporting a task as complete, verify:

- [ ] Raw files were not modified.
- [ ] Metadata sidecars exist and are valid.
- [ ] Metadata status is updated correctly.
- [ ] Generated notes follow `schema/templates/note.md`.
- [ ] Note titles are descriptive (less than 18 words, not single-word unless proper name).
- [ ] Note filenames match slugified titles (lowercase kebab-case).
- [ ] All categories exist in `categories.index.json`.
- [ ] All tags exist in `tags.index.json`.
- [ ] Every note includes provenance (`derived_from`).
- [ ] `generated_notes` is populated in metadata.
- [ ] Wikilinks resolve correctly.
- [ ] No schema, taxonomy, or validation violations remain.

---

## Failure Handling

- If processing fails, set metadata `status: failed` and populate `error` with details.
- Do not partially mark metadata as `processed` on failure.
- Do not delete raw files.
- Do not delete failed metadata — it may be retried after correction.
- Ask for clarification only when schema interpretation or source intent is genuinely ambiguous. Otherwise, follow the rules and proceed.

---

## Output Expectations

- Make minimal correct changes. Do not rewrite files unnecessarily.
- When reporting completion, state what changed, what was validated, and what remains.
- Avoid conversational or temporary observations in generated notes.
- Generated notes must be reusable, factual, and understandable in isolation.
