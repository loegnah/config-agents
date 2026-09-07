---
name: git-lgnh
description: "Git automation and repository maintenance workflow runner."
---

# GIT-LGNH Skill Runner

Execute git automation and repository maintenance workflows ONLY when explicitly invoked with `/git-lgnh <keyword>`.

## Keyword Subcommand Matcher

When the user inputs `/git-lgnh <keyword>`, match `<keyword>` against the keywords below to execute the corresponding catalog workflow:

**If invoked without a keyword (`/git-lgnh` only)**: Print the table below (available keywords and workflows), prompt the user to choose a workflow, and exit. Do not read or execute any catalog files.

| Keywords                                                  | Target Workflow   | Catalog File                   |
| :-------------------------------------------------------- | :---------------- | :----------------------------- |
| `commit`, `commit-detail`, `ci`                           | Commit Detail     | `catalog/commit-detail.md`     |
| `branch`, `clear-gone`, `gone`, `branch-clean`            | Branch Clear Gone | `catalog/branch-clear-gone.md` |
| `conflict`, `resolve`, `resolve-conflict`, `fix-conflict` | Resolve Conflict  | `catalog/resolve-conflict.md`  |
| `rebase`, `rebase-worktrees`, `sync-worktrees`            | Rebase Worktrees  | `catalog/rebase-worktrees.md`  |

## Execution Procedure

1. **Locate Skill Directory**:
   - Find the resolved installation directory of this `skills/git-lgnh/SKILL.md` file.
2. **Read Catalog Instruction**:
   - Read `<git_lgnh_skill_dir>/catalog/<filename>.md` matching the `<keyword>` using `read`.
3. **Execute Steps**:
   - Follow the instructions in the catalog file sequentially.

## No-Keyword Behavior

If `<keyword>` is empty (`/git-lgnh` only), do not execute any catalog. Print the Keyword Subcommand Matcher table above to guide available commands.
