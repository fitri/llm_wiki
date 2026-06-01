# Metadata Tagging Skill

## Purpose

Generate metadata sidecar files for raw materials without modifying raw file content.

## Workflow

1. Scan `raw/` for files without corresponding `filename.ext.metadata.json`.
2. **Normalize the raw filename** to lowercase kebab-case. Rename the raw file if its name does not follow the naming convention. Do not modify file content.
3. For each new file, generate a metadata sidecar using the template at `schema/templates/file.metadata.json`. The metadata filename must match the normalized raw filename.
4. Classify `content_type` based on file extension and content analysis.
5. Classify `source_type` based on content, filename, and context.
6. Assign an immutable ID using the ID generation standard.
7. Set `status` to `detected`.
8. Set `created_at` to current UTC ISO 8601 timestamp.
9. Write the metadata sidecar file and mark the raw file as ready for processing.

## Rules

- Never modify original raw file content.
- Raw filenames must be normalized to lowercase kebab-case before metadata generation.
- Generate metadata only — do not extract or process content.
- Metadata sidecar filename uses the normalized raw filename: `filename.ext.metadata.json`.
- Do not process files that already have a metadata sidecar unless the status is `failed`.
- If a file cannot be classified, set `content_type` and `source_type` to `other`.

## Output

One metadata sidecar file per raw file:

```text
raw/filename.ext.metadata.json
```
