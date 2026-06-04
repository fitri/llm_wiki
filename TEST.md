# Vault Pipeline Test

## Overview

This test validates the full vault pipeline from source drop through metadata generation, note creation, taxonomy management, and health check.

**Invocation:** Run `opencode run "run the test"` or any prompt that activates this file.

**What it tests:**
- Filename normalization (ALL_CAPS, MixedCase, spaces, generic names)
- Content-aware file renaming (vague/short names → descriptive 4-12 word names)
- Metadata sidecar generation with correct types
- Atomic note generation following the note template
- Taxonomy index creation and validation
- Filename-title parity (slug matches filename)
- Health check report generation

---

## Phase 1: Backup Current Vault State

1. Create `.digest/test_backup/source/` and `.digest/test_backup/notes/`.
2. Move all files from `source/` (except `.gitkeep`) to `.digest/test_backup/source/`.
3. Move all files from `notes/` (except `.gitkeep`) to `.digest/test_backup/notes/`.
4. Verify both directories contain only `.gitkeep`. If any files remain, abort with error.

---

## Phase 2: Create Test Sources

Generate 5 source files in `source/` with realistic substantive content. Each file tests a specific naming convention:

### File 1: `AGENT-SYSTEM-DESIGN-NOTES.md`
- **Type:** text
- **Tests:** ALL_CAPS → lowercase kebab normalization. Name is already descriptive (4 words, note-title quality) so it should be kept after normalization to `agent-system-design-notes.md`, not renamed further.
- **Content:** Write 3-4 paragraphs about designing AI agent systems — responsibility-based architecture, skill loading patterns, quality assurance checklists.

### File 2: `My Research on transformer and attention.md`
- **Type:** text
- **Tests:** Mixed Case + spaces → renamed to a descriptive content-aware name
- **Content:** Write 3-4 paragraphs about transformer architecture, attention mechanisms, multi-head attention, self-attention.

### File 3: `clippings.md`
- **Type:** webclip (include ChatGPT-style frontmatter with `title`, `source`, `created`, `tags`)
- **Tests:** Generic single-word name → renamed to match webclip content
- **Content:** A ChatGPT conversation snippet about building an Obsidian knowledge management system with metadata-driven retrieval.

### File 4: `urls.txt`
- **Type:** links
- **Tests:** Generic single-word name → renamed to match link collection content
- **Content:** 3-4 URLs on separate lines related to a specific topic (e.g. self-hosted AI tools, cloud storage services).

### File 5: `lite setup guide.md`
- **Type:** text
- **Tests:** Short vague name + spaces → renamed to match detailed setup content
- **Content:** Write a step-by-step setup guide for a technical tool (e.g. installing and configuring a database, web server, or container runtime).

---

## Phase 3: Confirm and Run

1. List all 5 files created with their file sizes.
2. Ask: "5 test files ready. Run the vault pipeline? (yes/no)"
3. If **no** → go to Phase 5 Cleanup with the "restore" path, then exit.
4. If **yes** → invoke AGENTS.md to run the full pipeline: "process the vault" or "digest".

---

## Phase 4: Validate Results

### A. Note Count Report

Report to the user before detailed validation:

```
Pipeline Results
=================
Source files processed:  5/5
Notes created:           X
Notes published:         X
Metadata sidecars:       5/5
Health check:            pass/fail
```

- **Notes created:** count of `.md` files in `notes/` excluding `.gitkeep`, `categories.index.json`, `tags.index.json`
- **Notes published:** count of metadata sidecars in `source/` with `status: publish`

### B. Per-Source Validation (8 checks each)

For each of the 5 source files:

| # | Check | How to verify |
|---|-------|---------------|
| 1 | Metadata sidecar exists | `source/filename.ext.metadata.json` exists |
| 2 | Filename is lowercase kebab-case | Single case, hyphens only, no spaces/underscores |
| 3 | Status is publish | Read sidecar: `"status": "publish"` |
| 4 | Type is correct | Read sidecar: matches expected type (text/webclip/links) |
| 5 | Note inherited source ID | At least one note in `notes/` has matching `id` field |
| 6 | Slug matches filename | Note frontmatter `slug` equals filename without `.md` |
| 7 | Frontmatter fields | Note has: `id`, `slug`, `date`, `categories`, `tags`, `summary` |
| 8 | Body sections | Note has: `# Title`, `## Summary`, `## Core Concepts`, `## Details` |

### C. Taxonomy Validation (2 checks)

| # | Check | How to verify |
|---|-------|---------------|
| 9 | Categories exist | All categories used in notes are in `notes/categories.index.json` |
| 10 | Tags exist | All tags used in notes are in `notes/tags.index.json` |

### D. Health Check Validation (2 checks)

| # | Check | How to verify |
|---|-------|---------------|
| 11 | Report exists | `.digest/vault-health-check-report.md` is present |
| 12 | All checks pass | Report shows core checks (checks 1-7) passed |

### E. Naming Behavior Validation (5 checks)

| # | Source | Expected behavior |
|---|--------|-------------------|
| 13 | File 1 | Filename normalized to lowercase. Current name `AGENT-SYSTEM-DESIGN-NOTES` is already 4 words, descriptive, and carries meaning — kept after normalization. |
| 14 | File 2 | Renamed. Original `My Research on transformer and attention.md` is too conversational for a vault filename. |
| 15 | File 3 | Renamed. `clippings.md` is too generic — should be renamed to reflect ChatGPT/knowledge management content. |
| 16 | File 4 | Renamed. `urls.txt` is too generic — should be renamed to reflect the URL collection topic. |
| 17 | File 5 | Renamed. `lite setup guide.md` is vague — should be renamed to reflect the specific tool guide content. |

### F. Output Format

Print a summary:

```
Validation Results
==================
Source 1 (AGENT-SYSTEM-DESIGN-NOTES.md): 8/8 ✓  Renamed? kept
Source 2 (My Research on transformer and attention.md): 8/8 ✓  Renamed? yes
Source 3 (clippings.md): 8/8 ✓  Renamed? yes
Source 4 (urls.txt): 8/8 ✓  Renamed? yes
Source 5 (lite setup guide.md): 8/8 ✓  Renamed? yes
Taxonomy: 2/2 ✓
Health Check: 2/2 ✓
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
TOTAL: 44/44 ✓ — PASS
Notes Created: X | Notes Published: X
```

If any check fails, print `FAIL` instead of `PASS` and list the failing items.

---

## Phase 5: Cleanup

After validation completes, ask:

**"Test complete. Clean up test files and reset vault to empty? (yes/no)"**

### If yes — Reset to empty:
1. Delete all files from `source/` (keep `.gitkeep`).
2. Delete all files from `notes/` (keep `.gitkeep`).
3. Delete any files in `.digest/` created during this test run (e.g. `vault-health-check-report.md`).
4. Leave `.digest/test_backup/` intact on disk for manual restore.
5. Report: "Vault reset to empty. Backup preserved at .digest/test_backup/"

### If no — Restore original vault:
1. Delete all files from `source/` (keep `.gitkeep`).
2. Delete all files from `notes/` (keep `.gitkeep`).
3. Move everything from `.digest/test_backup/source/*` → `source/`.
4. Move everything from `.digest/test_backup/notes/*` → `notes/`.
5. Delete `.digest/test_backup/`.
6. Delete any files in `.digest/` created during this test run.
7. Report: "Vault restored to pre-test state."

---

*This file is standalone. No project file references it.*
