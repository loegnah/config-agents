# Commit Detail

Stage all changes and create a git commit.

## Execution Steps

1. **Lint & Typecheck Verification**
   - Run the appropriate lint and typecheck commands for the project environment before proceeding (e.g., `bun run check`, `npm run lint`, `npm run typecheck`, or equivalent scripts based on `package.json` / project config).
   - If any lint or typecheck errors occur, report the issue to the user immediately and **do not proceed** with the commit process. Stop here.

2. **Stage & Inspect (Single Run)**
   - Run `git add -A && git status -s && git diff --cached --stat`
   - If output is empty (working tree is clean), report to user and stop immediately.

3. **Create Commit Message**
   - Use English only.
   - Use a conventional commit prefix: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, etc.
   - Write a concise subject line (under 72 characters).
   - Add concise bullet points (`- `) only for complex changes that need context; skip body for standard/simple changes.

4. **Execute Commit**
   - Run `git commit -m "<subject>"` (or pass additional `-m "<bullet>"` flags if body is needed).
