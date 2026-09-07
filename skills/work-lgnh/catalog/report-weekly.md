# Report Weekly

Weekly work report generated from git commits.

## Arguments

- Date range (optional): Default is last week (Monday to Sunday).

## Execution Steps

1. **Determine Date Range & User**
   - Default range: Last week (Mon-Sun).
   - Get current user: `git log --format="%an" | head -1`.
   - Today's date: `date +%Y-%m-%d`.

2. **Collect Commits**
   - Query commits: `git log --author="<user>" --after="<start_date>" --before="<end_date_+1day>" --oneline --no-merges`
   - Inspect stats: `git log --author="<user>" --after="<start_date>" --before="<end_date_+1day>" --no-merges --stat`

3. **Group & Summarize Tasks**
   - Group related commits into topics.
   - Use clear, non-technical Korean concise noun phrases (명사형 종결).
   - Avoid conversational endings.

4. **Output Markdown File**
   - File path: `report-weekly-YYMMDD-YYMMDD.md` in project root.
   - Format: **Bold Topic** followed immediately by bulleted items (`- `). No heading tags (`#`).
