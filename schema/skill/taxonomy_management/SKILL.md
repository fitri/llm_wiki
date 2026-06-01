# Taxonomy Management Skill

## Purpose

Manage the controlled vocabulary for categories, tags, and aliases.

## Workflow

1. Read `schema/taxonomy/categories.index.json`.
2. Read `schema/taxonomy/tags.index.json`.
3. Read `schema/taxonomy/aliases.index.json`.
4. When assigning categories or tags, search existing values first.
5. Reuse existing values whenever possible.
6. Create new values only when no existing value captures the concept.
7. Update the appropriate index file when adding new values.
8. Check for and prevent duplicate entries.

## Rules

- Always search existing categories and tags before creating new ones.
- Categories should be broad domains. Expected count: 20-50.
- Tags represent specific concepts. Count may grow indefinitely.
- Taxonomy values use lowercase snake_case.
- Aliases map common terms to canonical values to prevent vocabulary drift.
- New category creation should be rare.
- Every category and tag entry must have a `name` and `description`.

## Output

Updated taxonomy index files:
- `schema/taxonomy/categories.index.json`
- `schema/taxonomy/tags.index.json`
- `schema/taxonomy/aliases.index.json`
