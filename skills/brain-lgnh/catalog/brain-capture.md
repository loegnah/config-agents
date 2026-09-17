# Brain Capture

Record newly discovered information, meeting notes, ideas, or knowledge into the second brain's raw directory (`~/note/brain/raw/`).

## When to Use

- When the user asks to save, record, remember, or capture notes/findings (e.g. "기록해줘", "메모해줘", "capture해줘", "알아둬").
- When capturing raw information before subsequent wiki processing.

## Prohibitions

- **NEVER** run `ingest` or `init` workflows.
- Only record the source document in `~/note/brain/raw/`.

## Storage & Naming Conventions

1. **Target Directory**: `~/note/brain/raw/`
2. **File Name**:
   - Concise name reflecting the core content.
   - Connect words with hyphens (`-`). No spaces or special characters.
   - Extension must be `.md`.
   - Example: `~/note/brain/raw/React-19-전환-계획-수립.md`

3. **Frontmatter**:
   - Must include `created` date at the top:
     ```yaml
     ---
     created: YYYY-MM-DD
     ---
     ```

4. **Document Body**:
   - Place `# Title` after frontmatter with normal spaces (no hyphens).
   - Organize content cleanly using headers, bullet lists, and code blocks.

## Execution Steps

1. **Extract Content**: Identify the core information, decisions, or notes from user input or context.
2. **Determine File Name**: Create a hyphen-separated filename under `~/note/brain/raw/`.
3. **Write Markdown**: Format with `created` frontmatter and structured body.
4. **Report**: Confirm creation with the file path and brief summary.
