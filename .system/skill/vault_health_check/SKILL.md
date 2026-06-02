# Vault Health Check Skill

## Purpose

Maintain vault integrity by detecting and reporting issues.

## Checks

- **Concept Gaps** — Detect `[[wikilinks]]` that point to non-existent notes. Classified as informational — not an error, as gaps are expected to be filled over time.
- **Pending Stubs** — Detect notes with `type: auto` and `status: pending` that are awaiting human content. Classified as a warning.
- **Missing Metadata** — Detect source files without corresponding `filename.ext.metadata.json`.
- **Orphan Assets** — Detect assets in `assets/` not referenced by any note.
- **Stalled Processing** — Detect metadata sidecars stuck at `status: digest` that have not progressed to `publish`.
- **Invalid Categories** — Detect notes using categories not in `categories.index.json`.
- **Invalid Tags** — Detect notes using tags not in `tags.index.json`.
- **Schema Violations** — Detect notes or metadata files that do not follow the defined schema.
- **Filename-Title Mismatch** — Detect notes where the `slug` field in frontmatter does not match the note filename (without `.md`).
- **Stale Notes** — Detect notes that have not been updated in a long time.
- **Taxonomy Drift** — Detect duplicate or near-duplicate category/tag entries.

## Auto Note Relaxations

Notes with `type: auto` and `status: pending` are exempt from certain checks:
- No 5-word minimum for naming (single-word titles are acceptable for concept stubs).
- Empty or placeholder content is allowed.
- However, template structure and slug parity must still pass.

## Filename Rules

- When organizing or renaming files during health check, use lowercase kebab-case.
- Attachments should use lowercase kebab-case when organized by AI.
- Source file content must not be modified.
- Source filenames may be normalized or renamed by AI when organizing the vault.

## Output

A report file summarizing all issues found and actions taken.
Suggested location: `.digest/vault-health-check-report.md`.
