# Commit Detail

Stage all changes and create a detailed git commit.

## Execution Steps

1. **Lint & Typecheck Verification**
   - Run the appropriate lint and typecheck commands for the project environment before proceeding (e.g., `bun run check`, `npm run lint`, `npm run typecheck`, or equivalent scripts based on `package.json` / project config).
   - If any lint or typecheck errors occur, report the issue to the user immediately and **do not proceed** with the commit process. Stop here.

2. **Collect Context**
   - Run `git branch --show-current`, `git status`, `git diff HEAD`, and `git log --oneline -10` to inspect working tree state and recent commits.

3. **Stage All Changes**
   - Run `git add -A`
   - Include all working-tree changes (staged, unstaged, unintended edits) — never selectively exclude or revert.

4. **Create Commit Message**
   - Use English only.
   - Use a conventional commit prefix: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, etc.
   - Write a concise subject line (under 72 characters).
   - Include a body with bullet points (`- `) for multiple changes or value-add details.
   - Skip body only for trivial single-line changes.
   - Separate subject from body with a blank line.

5. **Execute Commit**
   - Use heredoc for multiline formatting:
     ```bash
     git commit -m "$(cat <<'EOF'
     subject line here

     - bullet point 1
     - bullet point 2
     EOF
     )"
     ```
   - If working tree is clean, report to user and stop.
