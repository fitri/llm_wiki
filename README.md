# LLM Wiki

AI-assisted knowledge warehouse built on Obsidian. Drop source files into `source/` — agents normalize, tag, and convert them into atomic, searchable knowledge notes.

## How It Works

```
Human drops file into source/
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

Four AI agents operate on a controlled vocabulary system. Source content is never modified. Every note traces back to its source.

## Vault Structure

```
wiki/
├── AGENTS.md → .system/AGENTS.md  ← operating guide for all agents (symlink)
├── source/                  ← human ingestion zone (content ignored by git)
├── notes/                   ← flat knowledge repository (content ignored by git)
├── .digest/              ← temporary AI workspace
├── archive/                 ← deprecated notes
├── assets/                  ← images, diagrams, attachments
└── .system/
    ├── AGENTS.md            ← operating guide & agent definitions (real file)
    ├── templates/
    │   ├── note.md          ← knowledge note template
    │   ├── file.metadata.json
    │   ├── categories.example.json
    │   └── tags.example.json
    └── skill/
        ├── metadata_tagging/SKILL.md
        ├── source_to_note/SKILL.md
        ├── taxonomy_management/SKILL.md
        └── vault_health_check/SKILL.md
```

## Design Principles

- **Human effort is minimized** — drop files, AI does the rest
- **Metadata drives organization** — no folder hierarchies, everything is flat
- **Controlled vocabulary** — categories and tags are predefined, preventing drift
- **Provenance is preserved** — every note links back to its original source
- **Search-first** — retrieval is through metadata and full-text search, not folders
- **Scales to 100,000+ notes** — designed without structural redesign needed

## Agents

| Agent | Responsibility |
|---|---|
| **Metadata Agent** | Scan source, normalize filenames, generate metadata sidecars, classify content |
| **Note Generation Agent** | Read source, extract knowledge, generate atomic notes, assign taxonomy |
| **Taxonomy Agent** | Manage categories and tags; prevent vocabulary drift |
| **Health Check Agent** | Detect broken links, orphan assets, invalid taxonomy, schema violations |

## Getting Started

1. Open this vault in Obsidian.
2. Drop files (PDF, TXT, URLs, images, chat exports) into `source/`.
3. Run opencode with an activation prompt:
   ```
   opencode run "digest"
   ```
   Or any prompt like "process the vault", "run the agents" — OpenCode reads
   `AGENTS.md` and orchestrates the full pipeline automatically.

No manual tagging, no frontmatter editing, no folder organization required.
