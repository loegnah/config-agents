# Brain Query

Search and answer questions using information stored in the second brain (`~/note/brain`).

## When to Use

- When the user asks questions about notes, wiki, or records in their second brain (e.g. "브레인에서 찾아줘", "wiki 확인해줘", "검색해줘").
- When synthesizing information across multiple brain pages.

## Prohibitions

- **NEVER** run `ingest` or `init` workflows.
- `refs/` directory is strictly read-only (do not modify or delete).

## Execution Steps

1. **Catalog Check**: Read `~/note/brain/index.md` first to identify candidate pages by category.
2. **Explore & Cross-Reference**:
   - Follow `[[wikilink]]` references in candidate pages to gather context.
   - Grep in `~/note/brain/wiki/` or `~/note/brain/refs/` when needed to locate specific keywords.
3. **Synthesize Answer**:
   - Compose a clear response in Korean based on collected pages.
   - Cite source pages using `[[wikilink]]` format (e.g. `[[React-19-전환-계획]]`).
   - If information is conflicting, state it clearly.
   - If information is missing or insufficient, state that honestly—never speculate.
4. **Suggest Saving (Optional)**:
   - If the answer has lasting value (analysis, comparison, summary), ask user confirmation before saving to `~/note/brain/wiki/analyses/<제목>.md`.
   - When saved, frontmatter includes:
     ```yaml
     ---
     type: analysis
     created: YYYY-MM-DD
     updated: YYYY-MM-DD
     based_on:
       - "[[Page1]]"
       - "[[Page2]]"
     ---
     ```
