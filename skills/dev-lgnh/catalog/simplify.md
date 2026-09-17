# Code Simplify

Simplify recently changed code without behavior change by delegating the refactoring task to a subagent with summarized session context.

## Arguments

- `scope` (optional): Target directory or file pattern.

## Constraints

- **절대로 `git add`, `git commit` 등 git 상태를 변경하거나 커밋하는 작업을 수행하지 마세요.**
- Git repository의 상태(stage, commit 등)를 절대로 수정하지 말고, 오직 대상 파일의 코드 단순화 작업만 수행하세요.
- **서브에이전트(Subagent) 필수 사용**: 실제 코드 수정 및 단순화 작업은 `task` 도구를 통해 서브에이전트를 생성하여 위임합니다.

## Execution Steps

1. **Detect Target Scope & Changes**
   - If scope provided, set `$TARGET_SCOPE`.
   - If scope omitted:
     - Check uncommitted changes: `git diff --name-only` + `git diff --name-only --cached`.
     - If none, check last commit: `git diff --name-only HEAD~1...HEAD`.
   - If no target files found, inform user and exit.

2. **Summarize Session Context**
   - Collect context from the ongoing session:
     - Identify what feature/fix was recently being worked on and the intent behind the changes.
     - Obtain diff details (`git diff` or `git show HEAD`) for the target files.
   - Draft a clear summary containing:
     - Core objective of the session/changes.
     - Target file list and detailed diff context.
     - Behavioral invariants to preserve.

3. **Delegate Simplification to Subagent**
   - Spawn a subagent via the `task` tool:
     - `context`: Pass the session summary, target file paths, diff overview, and strict non-git modification constraints.
     - `tasks`: Instruct subagent to refactor target files for readability and simplicity while preserving exact behavior.
   - Subagent Guidelines:
     - Retain exact behavior, API contracts, and return values.
     - Follow project coding standards (`CLAUDE.md`, etc.).
     - Prioritize readability over brevity. Avoid extreme single-line rewrites or nested ternaries.
     - DO NOT perform `git add` or `git commit`.

4. **Verify & Report Results**
   - Review subagent's changes (`git diff`).
   - Summarize modified files and applied refinements in 1-2 lines.
   - Recommend testing and reviewing `git diff`.
