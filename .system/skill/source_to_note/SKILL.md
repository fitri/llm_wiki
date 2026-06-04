# Source to Note Skill

## Purpose

Convert source material into atomic knowledge notes.

## Workflow

1. Read a source file from `source/` that has `status: new`.
2. Update metadata status to `digest`.
3. Extract and analyze content from the source file.
4. Determine knowledge density and decide note count.
5. Generate one or more knowledge notes following the atomic note rules.
6. Derive each note filename from its title by slugifying the title to lowercase kebab-case.
7. Assign categories and tags from the taxonomy indexes.
8. Generate a concise summary for each note.
9. Verify filename-title parity for every generated note — ensure the `slug` field matches the note filename (without `.md`).
10. Set metadata status to `publish`.

## Atomic Note Rules

A knowledge note should contain one reusable idea.

**Create a separate note when the source contains:**
- A distinct concept.
- A distinct process or workflow.
- A distinct decision or conclusion.
- A distinct rule, principle, or best practice.
- A distinct technical reference.
- A distinct troubleshooting pattern.
- A distinct comparison between ideas, tools, or methods.
- A distinct reusable question-and-answer pair.

**Do not create separate notes for:**
- Minor details.
- Repeated statements.
- Temporary observations.
- Personal commentary with no reusable value.
- Source summaries that do not introduce reusable knowledge.
- Content that only makes sense inside the original document.

Each note must be understandable without reading the full original source.

Each note should answer at least one of these questions:
1. What is this?
2. How does this work?
3. Why does this matter?
4. When should this be used?
5. What problem does this solve?
6. What should be remembered from this?

Prefer fewer high-quality reusable notes over many shallow notes. Do not force-split a single source into multiple notes. Only split when topics are truly too big and diverse to fit in one note.

Titles must follow `.system/templates/naming-conventions.md`: 4-12 meaningful words, lead with dominant concept. Run the meaningfulness self-check (Check A/B/C in naming-conventions.md) on every title before writing. The title alone should inform what the note is about.

## Chat Processing

When the source type is `chats` or `webclip`, or when a `links` or `text` source contains conversational content between a human user and an LLM, apply the following processing rules.

### Format Detection

Identify the chat format from the source content:

- **Markdown / plain text** — Scan for speaker labels such as `User:`, `Human:`, `You:`, `ChatGPT:`, `Assistant:`, `AI:`, `LLM:` to identify speaker turns.
- **JSON exports** — Detect standard ChatGPT export structures with `mapping`, `message`, and `author.role` fields. Extract conversation turns sequentially.
- **Obsidian webclipper** — Treat frontmatter-wrapped content as markdown chat; parse speaker labels from the body.
- **Links** — If a `links` type file or any chat source contains URLs pointing to shareable conversations (e.g. `chatgpt.com/share/...`), fetch the linked content.

### Link Following

When the source contains conversation URLs:

1. Scan the source for URLs pointing to shareable conversations.
2. Fetch each URL's content using the available web fetch tools.
3. Extract the conversation text from the fetched response.
4. Save extracted content to `.digest/chat_extracts/` as an intermediate artifact.
5. Use the extracted text as the primary knowledge source for note generation.
6. If a URL cannot be fetched (auth gate, JS-rendered, inaccessible), note the failure in `.digest/` and skip that URL.

### Speaker Turn Identification

Detect speaker turns by scanning for known boundaries:

| Speaker | Labels |
|---------|--------|
| **User** | `User:`, `Human:`, `You:` |
| **Assistant/AI** | `ChatGPT:`, `Assistant:`, `AI:`, `LLM:` |
| **JSON** | `"author": {"role": "user"}` or `"author": {"role": "assistant"}` |

Treat assistant responses as the primary knowledge source. Use user prompts as context for understanding intent.

### Discard Rules

Strip the following before generating notes:

- Greetings and closings with no reusable value.
- Offers of further help or follow-up questions from the assistant.
- Meta-commentary about the conversation itself.
- User-only turns that contain only questions with no standalone knowledge.
- Repeated or near-identical statements across turns.

### Multi-Turn Handling

- When a topic spans multiple conversation turns, combine the relevant content into a single concept extraction. Do not create separate notes per turn.
- When a later LLM response corrects or refines an earlier statement, prefer the later version.
- When the same concept is explained multiple times (e.g. rephrased or summarized), extract once. Do not create duplicate notes.

### Knowledge Extraction Priority

1. **Assistant responses** — The primary source of factual, reusable knowledge.
2. **User prompts** — Use for context only. Do not generate standalone notes from user questions alone.
3. **Output** — Atomic notes that are self-contained. A reader should understand the note without seeing the original conversation.

## Filename Rules

- Generated notes must use lowercase kebab-case filenames.
- Notes must use `.md` extension.
- Note filenames must match note titles. Derive the filename by slugifying the title to lowercase kebab-case.

Example: A note titled "How Transformer Attention Works" becomes `how-transformer-attention-works.md`.

## Guiding Principle

Create knowledge details notes, not summaries.

## Template

All generated notes must follow `.system/templates/note.md`.

## Output

Knowledge notes in `notes/` and updated metadata sidecar files.
