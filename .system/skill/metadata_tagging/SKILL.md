# Metadata Tagging Skill

## Purpose

Generate metadata sidecar files for source materials without modifying source file content.

## Workflow

1. Scan `source/` for files without corresponding `filename.ext.metadata.json`.
2. **Read and understand the source material.** Determine what the content is about. Derive a clear 4-12 word name that describes the source content, following the rename rules in `.system/templates/naming-conventions.md`. Lead with the most dominant concept; follow with other relevant concepts if space allows. If the current filename is already a good descriptive name within 4-12 words, keep it after normalization. Otherwise rename. 3 words or fewer always triggers a rename. This understanding also informs type classification.
3. **Normalize the source filename** to lowercase kebab-case. Do not modify file content.
4. For each new file, generate a metadata sidecar using the template at `.system/templates/file.metadata.json`. The metadata filename must match the normalized source filename.
5. Classify `type` based on file extension and content.
6. Assign an immutable ID using the ID generation standard.
7. Set `status` to `new`.
8. Set `created_at` to current Unix epoch seconds.
9. Write the metadata sidecar file and mark the source file as ready for processing.

## Self-Check

Before finalizing the renamed source filename, run the meaningfulness checks in `.system/templates/naming-conventions.md`:
- Check A: No banned patterns (hex, UUID, numeric-dominant, single-letter words).
- Check B: ≥ 3 non-stop words.
- Check C: ≥ 50% meaningful word ratio.

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
