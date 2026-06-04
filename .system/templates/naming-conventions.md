# Naming Conventions

Applies to all vault-managed filenames in `source/`, `notes/`, `assets/`, `archive/`, and `.digest/`.

## Rules

1. **Lowercase kebab-case** — lowercase letters, hyphens between words.
2. **No spaces, underscores, or special characters** (except file extension dot).
3. **4–12 words** — inclusive. 3 words or fewer always triggers a rename.
4. **All words must carry meaning** related to the content. No filler words, hex strings, UUIDs, IDs, or random characters.
5. **Lead with the most dominant concept** in the content; follow with other relevant concepts if space allows within the 4-12 word limit.
6. **Source filenames may be renamed by AI.** Source content must not be modified.
7. **Generated note filenames must match the note title.** Do not use the source file name as the note filename.
8. **Maximum 255 characters** — filesystem hard limit. File basename (without extension) must not exceed 255 characters. Shorten by removing the least essential words if exceeded.

## Source-Note Relationship

Source filename and note filenames are independent. They are connected by the shared `id` field — not by filename similarity.

| Relationship | Type | Connection |
|---|---|---|
| Source ↔ Metadata sidecar | One-to-one | Same filename: `topic-name.md` ↔ `topic-name.md.metadata.json` |
| Source → Notes | One-to-one or one-to-many | Shared `id` in sidecar and all derived notes |

A source named `self-hosted-ai-tools-comparison.md` can generate notes titled `running-local-models-with-ollama.md` and `comparing-cloud-and-local-inference-costs.md`. Both inherit the same `id` from the source. The filenames are derived from their own note titles, not the source filename.

## Rename Decision Flow

1. Read the source content to understand its topic.
2. Derive a clear 4-12 word name that describes the source content. Lead with the most dominant concept.
3. Compare the derived name against the current filename:
   - If current name is already 4-12 words, descriptive, and carries meaning → keep it (after mechanical normalization).
   - Otherwise → rename the file.
   - **3 words or fewer always triggers a rename**, even if the name is descriptive.
4. Run the meaningfulness self-check before finalizing.
5. Normalize to lowercase kebab-case.
6. **Generate the ID only after the name is final and normalized.**

## Meaningfulness Self-Check

Before finalizing any filename or note title, verify all three checks:

### Check A: Banned Pattern Detector
Reject if any word matches:
- Hex-like strings (e.g. `a7f3b2c1`, `deadbeef`)
- UUID / GUID patterns
- Numeric-dominant words (more than 50% digits)
- Single-letter words (e.g. `x`, `a`, `b`)

**PASS** if no banned patterns found.

### Check B: Minimum Meaningful Word Count
Count total words. Subtract stop words (see list below). Meaningful words must be ≥ 3.

### Check C: Stop-Word Ratio
```text
meaningful_ratio = (total_words - stop_words) / total_words
```
- ≥ 50% → PASS
- < 50% → FAIL (too many filter words)

### Check D: Character Limit
- Count characters in the basename (excluding file extension).
- ≤ 255 → PASS
- > 255 → FAIL — filename will be truncated by the filesystem, destroying meaning.

## Stop Words

The following words do not count as meaningful:

```
the, a, an, my, some, about, into, with, for, and, or, of, in, on, to,
is, are, was, were, be, been, this, that, these, it, its, id, note,
notes, guide, how, what, when, why, etc, can, will, should, would
```

## Examples

| Good | Bad | Why Bad |
|---|---|---|
| `how-transformer-attention-works-internally` (5 words) | `transformer` (1 word) | Too short, too vague |
| `rclone-virtual-backends-explained` (4 words) | `rclone-virtual-backends` (3 words) | Under 4-word minimum |
| `container-orchestration-with-kubernetes` (4 words) | `a-container-orchestration-guide-using-kubernetes` (6 words, 3 stops) | 50% ratio, padded with filler |
| `self-hosted-ai-tools-comparison` (4 words) | `AGENT-SYSTEM-DESIGN` (3 words, ALL_CAPS) | Under minimum, wrong case |
| `database-installation-and-configuration-guide` (5 words) | `lite setup guide` (3 words, spaces) | Under minimum, spaces |
| `transformer-attention-mechanism-diagram` (4 words) | `urls` (1 word) | Not descriptive |
| `kubernetes-container-orchestration-explained` (4 words) | `a7f3b2c1-guide` (contains hex) | Banned pattern (Check A) |
