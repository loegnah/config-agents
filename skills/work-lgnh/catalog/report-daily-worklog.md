# Report Daily Worklog

Daily one-line git commit summary by date.

## Arguments

- `authors`: Comma or space separated git author names. Omit to auto-detect via `git log --format="%an" | head -1`.

## Execution Steps

1. **Determine Date Range**
   - Default: Current week (Monday to today).
   - If user mentions "last week" / "저번주" / "지난주", use that week (Mon-Sun).
   - Today's date: `date +%Y-%m-%d`.

2. **Collect Commits**
   - Query commits for all authors:
     ```bash
     git log --author="name1\|name2" --after="<start>" --before="<end>" --pretty=format:"%ad | %s" --date=short --all
     ```

3. **Format Daily Summary**
   - Group commits by date (exclude merge commits).
   - Summarize daily tasks cleanly separated by `/` (never use commas).
   - One line per date in Korean. Format: `**M/D (요일)**`
   - Omit weekend dates if no commits.
   - Output directly as text (do not write to file).
