# Taxonomy Management Skill

## Purpose

Manage the controlled vocabulary for categories and tags.

## Workflow

1. Read `.system/templates/categories.example.json` and `.system/templates/tags.example.json` for structure guidance.
2. Read `notes/categories.index.json` and `notes/tags.index.json` for existing runtime values. Create empty files if they do not exist.
3. When assigning categories or tags, search existing values first.
4. Reuse existing values whenever possible.
5. Create new values only when no existing value captures the concept.
6. Append new values to the appropriate runtime index file in `notes/`.
7. Check for and prevent duplicate entries.

## Rules

- Always search existing categories and tags before creating new ones.
- Categories should be broad domains. Expected count: 20-50.
- Tags represent specific concepts. Count may grow indefinitely.
- Taxonomy values use lowercase snake_case.
- New category creation should be rare.
- Every category and tag entry must have a `name` and `description`.

## Output

Updated runtime taxonomy index files:
- `notes/categories.index.json`
- `notes/tags.index.json`
