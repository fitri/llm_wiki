# LLM Wiki

AI-assisted knowledge warehouse built on Obsidian. Drop raw files into `raw/` — agents normalize, tag, and convert them into atomic, searchable knowledge notes.

## How It Works

```
Human drops file into raw/
        │
        ▼
Metadata Agent       →  filename.ext.metadata.json
        │
        ▼
Note Generation Agent  →  notes/*.md (atomic knowledge notes)
        │
        ▼
Taxonomy Agent       →  validates categories & tags
        │
        ▼
Health Check Agent   →  verifies metadata, titles, filenames, links
```

Four AI agents operate on a controlled vocabulary system. Raw content is never modified. Every note traces back to its source.

## Vault Structure

```
wiki/
├── instructions.md          ← master operating guide for all agents
├── plan.md                  ← master implementation plan
├── raw/                     ← human ingestion zone (content ignored by git)
├── notes/                   ← flat knowledge repository (content ignored by git)
├── processing/              ← temporary AI workspace
├── archive/                 ← deprecated notes
├── assets/                  ← images, diagrams, attachments
└── schema/
    ├── agents.md            ← agent definitions & responsibilities
    ├── instructions.md      ← pointer to root instructions.md
    ├── templates/
    │   ├── note.md          ← knowledge note template
    │   └── file.metadata.json
    ├── taxonomy/
    │   ├── categories.index.json
    │   ├── tags.index.json
    │   └── aliases.index.json
    └── skill/
        ├── metadata_tagging/SKILL.md
        ├── raw_to_note/SKILL.md
        ├── taxonomy_management/SKILL.md
        └── vault_health_check/SKILL.md
```

## Design Principles

- **Human effort is minimized** — drop files, AI does the rest
- **Metadata drives organization** — no folder hierarchies, everything is flat
- **Controlled vocabulary** — categories and tags are predefined, preventing drift
- **Provenance is preserved** — every note links back to its original raw source
- **Search-first** — retrieval is through metadata and full-text search, not folders
- **Scales to 100,000+ notes** — designed without structural redesign needed

## Agents

| Agent | Responsibility |
|---|---|
| **Metadata Agent** | Scan raw, normalize filenames, generate metadata sidecars, classify content |
| **Note Generation Agent** | Read raw, extract knowledge, generate atomic notes, assign taxonomy |
| **Taxonomy Agent** | Manage categories, tags, and aliases; prevent vocabulary drift |
| **Health Check Agent** | Detect broken links, orphan assets, invalid taxonomy, schema violations |

## Getting Started

1. Open this vault in Obsidian.
2. Drop files (PDF, TXT, URLs, images, chat exports) into `raw/`.
3. Run the metadata agent to generate sidecars.
4. Run the note generation agent to create knowledge notes.
5. Run the health check agent to validate output.

No manual tagging, no frontmatter editing, no folder organization required.
