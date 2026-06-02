# Metadata Tagging Skill

## Purpose

Generate metadata sidecar files for source materials without modifying source file content.

## Workflow

1. Scan `source/` for files without corresponding `filename.ext.metadata.json`.
2. **Read and understand the source material.** Determine what the content is about. If the current filename is too short, vague, or underspecified for the content, rename it to a descriptive 5–15 word lowercase kebab-case name. This understanding also informs type classification.
3. **Normalize the source filename** to lowercase kebab-case. Rename the source file if its name does not follow the naming convention. Do not modify file content.
4. For each new file, generate a metadata sidecar using the template at `.system/templates/file.metadata.json`. The metadata filename must match the normalized source filename.
5. Classify `type` based on file extension and content.
6. Assign an immutable ID using the ID generation standard.
7. Set `status` to `new`.
8. Set `created_at` to current Unix epoch seconds.
9. Write the metadata sidecar file and mark the source file as ready for processing.

## Rules

- Never modify original source file content.
- Source filenames must be normalized to lowercase kebab-case before metadata generation.
- Generate metadata only — do not extract or process content.
- Metadata sidecar filename uses the normalized source filename: `filename.ext.metadata.json`.
- Do not process files that already have a metadata sidecar unless the status is `new`.
- If a file cannot be classified, set `type` to `other`.

## Output

One metadata sidecar file per source file:

```text
source/filename.ext.metadata.json
```
