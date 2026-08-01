# Code Simplify

Simplify recently changed code without behavior change.

## Arguments

- `scope` (optional): Target directory or file pattern.

## Constraints

- **절대로 `git add`, `git commit` 등 git 상태를 변경하거나 커밋하는 작업을 수행하지 마세요.**
- Git repository의 상태(stage, commit 등)를 절대로 수정하지 말고, 오직 대상 파일의 코드 단순화 작업만 수행하세요.

## Execution Steps

1. **Detect Target Scope**
   - If scope provided, set `$TARGET_SCOPE`.
   - If scope omitted:
     - Check uncommitted changes: `git diff --name-only` + `git diff --name-only --cached`.
     - If none, check last commit: `git diff --name-only HEAD~1...HEAD`.
   - If no files found, inform user and exit.

2. **Simplify & Refine Code**
   - Retain exact behavior, API contracts, and return values.
   - Follow project coding standards (`CLAUDE.md`, etc.).
   - Prioritize readability over brevity. Avoid extreme single-line rewrites or nested ternaries.

3. **Report Results**
   - Summarize changed files and applied refinements in 1-2 lines.
   - Recommend testing and checking `git diff`.
