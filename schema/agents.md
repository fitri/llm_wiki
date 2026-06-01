# Agent Definitions

## Metadata Agent

**Responsibilities:**

- Scan `raw/`
- Detect new files
- Generate metadata
- Classify `content_type`
- Classify `source_type`

**Output:** `filename.ext.metadata.json`

---

## Note Generation Agent

**Responsibilities:**

- Read raw material
- Read metadata
- Extract content
- Determine knowledge density
- Decide note count
- Generate notes
- Assign categories
- Assign tags
- Generate summaries
- Create links
- Update `generated_notes`

**Output:** `notes/*.md`

---

## Taxonomy Agent

**Responsibilities:**

- Manage categories index
- Manage tags index
- Manage aliases
- Prevent duplicates
- Normalize vocabulary

---

## Health Check Agent

**Responsibilities:**

- Detect orphan assets
- Detect broken links
- Detect missing metadata
- Detect failed processing
- Detect stale notes
- Detect taxonomy drift
