# Vault Health Check Skill

## Purpose

Maintain vault integrity by detecting and reporting issues.

## Checks

- **Broken Links** — Detect `[[wikilinks]]` that point to non-existent notes.
- **Missing Metadata** — Detect source files without corresponding `filename.ext.metadata.json`.
- **Orphan Assets** — Detect assets in `assets/` not referenced by any note.
- **Stalled Processing** — Detect metadata sidecars stuck at `status: digest` that have not progressed to `publish`.
- **Invalid Categories** — Detect notes using categories not in `categories.index.json`.
- **Invalid Tags** — Detect notes using tags not in `tags.index.json`.
- **YAML Validity** — For every `.md` file in `notes/`, verify the frontmatter delimited by `---` is valid, parseable YAML. Notes with broken YAML are reported with the parse error. No further checks are run on that note.
- **Frontmatter Field Presence** — Every note must have all 6 required fields: `id`, `slug`, `date`, `categories`, `tags`, `summary`. Missing fields are reported. Run after YAML is successfully parsed.
- **Frontmatter Field Non-Empty** — Every field must have a non-empty value. Strings (`categories`, `id`, `slug`, `date`, `summary`) must not be `""` or whitespace-only. `tags` (list) must have ≥ 1 entry. Entries must not be empty.
- **Frontmatter Field Types** — `categories`, `id`, `slug`, `date`, `summary` must be strings. `tags` must be a list. Wrong types are flagged.
- **Duplicate IDs** — No two notes may share the same `id`. Report conflicting filenames and the shared ID.
- **Body Has Title** — Every note must contain a `# <title>` heading on the first content line after the closing `---`. Missing or empty heading text is flagged.
- **Body Has TL;DR** — Every note must contain a `## ✦ TL;DR` heading with at least one non-empty bullet.
- **Body Has No Empty Sections** — No section heading followed immediately by another heading with no content between. No section contains `N/A`, placeholder text, or filler phrases (`"Great question!"`, `"Certainly!"`, `"Sure!"`, `"I'd be happy to"`).
- **Code Block Language Tags** — Every fenced code block (````) must have a language identifier immediately after the opening backticks. Flag bare ` ``` ` with no language tag (e.g. ` ```\n` without `python`, `bash`, etc.).
- **Filename-Title Mismatch** — Detect notes where the `slug` field in frontmatter does not match the note filename (without `.md`).
- **Stale Notes** — Detect notes that have not been updated in a long time.
- **Taxonomy Drift** — Detect duplicate or near-duplicate category/tag entries.
- **Naming Convention Violations** — Verify source filenames follow the rules in `.system/templates/naming-conventions.md`:
  - Lowercase kebab-case format.
  - 4-12 words.
  - Passes meaningfulness checks (A: no banned patterns, B: ≥ 3 non-stop words, C: ≥ 50% meaningful ratio). Use the stop-word list in naming-conventions.md for checks B and C.
  - Check D: ≤ 255 characters (basename, excluding extension).

## Filename Rules

- When organizing or renaming files during health check, use lowercase kebab-case.
- Attachments should use lowercase kebab-case when organized by AI.
- Source file content must not be modified.
- Source filenames may be normalized or renamed by AI when organizing the vault.

## Output

A report file summarizing all issues found and actions taken.
Suggested location: `.digest/vault-health-check-report.md`.
