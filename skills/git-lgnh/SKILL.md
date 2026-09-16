---
name: git-lgnh
description: "Git automation and repository maintenance workflow runner."
---

# GIT-LGNH Skill Runner

Execute git automation and repository maintenance workflows when invoked with `/git-lgnh <keyword>` or matching git workflow intent.

## Rules

- NEVER run `glob`, `grep`, or directory listings to explore or verify catalog files.
- Read the exact relative file path (`catalog/<filename>.md`) directly.

## Workflow Dispatcher

Match the user's keyword or intent against the workflows below:

| Keywords / Intent                                         | Target Workflow   | Catalog File                   |
| :-------------------------------------------------------- | :---------------- | :----------------------------- |
| `commit`, `commit-detail`, `ci`                           | Commit Detail     | `catalog/commit-detail.md`     |
| `branch`, `clear-gone`, `gone`, `branch-clean`            | Branch Clear Gone | `catalog/branch-clear-gone.md` |
| `conflict`, `resolve`, `resolve-conflict`, `fix-conflict` | Resolve Conflict  | `catalog/resolve-conflict.md`  |
| `rebase`, `rebase-worktrees`, `sync-worktrees`            | Rebase Worktrees  | `catalog/rebase-worktrees.md`  |

## Execution Procedure

1. **Match Workflow**: Identify the catalog file from the table above.
2. **Read Catalog Directly**: Read `catalog/<filename>.md` directly using relative path.
3. **Execute Steps**: Follow the instructions in the catalog file sequentially.

## Fallback / No-Keyword Behavior

If invoked without a keyword (`/git-lgnh` only) or if no workflow matches:

- Print the table above to guide available commands and ask the user to choose.
- Do not read any catalog files or inspect directories.
