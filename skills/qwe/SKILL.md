---
name: qwe
description: "Run user-driven qwe workflows explicitly invoked via `/qwe <keyword>`."
---

# QWE Skill Runner

Execute custom developer and work reporting workflows ONLY when explicitly invoked with `/qwe <keyword>`.

## Keyword Subcommand Matcher

When the user inputs `/qwe <keyword>`, match `<keyword>` against the keywords below to execute the corresponding catalog workflow:

**키워드 없이 `/qwe`만 입력된 경우**: 아래 표(사용 가능한 키워드와 워크플로 목록)를 그대로 출력하고, 실행할 워크플로를 선택해 달라고 안내한 뒤 종료한다. 카탈로그 파일을 읽거나 실행하지 않는다.

| Keywords                                                     | Target Workflow      | Catalog File                      |
| :----------------------------------------------------------- | :------------------- | :-------------------------------- |
| `commit`, `commit-detail`, `ci`                              | Commit Detail        | `catalog/commit-detail.md`        |
| `branch`, `clear-gone`, `gone`, `branch-clean`               | Branch Clear Gone    | `catalog/branch-clear-gone.md`    |
| `simplify`, `code-simplify`, `clean-code`, `refactor-simple` | Code Simplify        | `catalog/code-simplify.md`        |
| `rebase`, `rebase-worktrees`, `sync-worktrees`               | Rebase Worktrees     | `catalog/rebase-worktrees.md`     |
| `conflict`, `resolve`, `resolve-conflict`, `fix-conflict`    | Resolve Conflict     | `catalog/resolve-conflict.md`     |
| `daily`, `worklog`, `report-daily`, `daily-log`              | Daily Worklog Report | `catalog/report-daily-worklog.md` |
| `weekly`, `report-weekly`, `weekly-report`                   | Weekly Work Report   | `catalog/report-weekly.md`        |

## Execution Procedure

1. **Locate Skill Directory**:
   - Find the resolved installation directory of this `skills/qwe/SKILL.md` file.
2. **Read Catalog Instruction**:
   - Read `<qwe_skill_dir>/catalog/<filename>.md` matching the `<keyword>` using `read`.
3. **Execute Steps**:
   - Follow the instructions in the catalog file sequentially.

## No-Keyword Behavior

`<keyword>`가 비어 있으면(`/qwe`만 입력) 카탈로그를 실행하지 말고, 위 Keyword Subcommand Matcher 표를 출력해 사용 가능한 명령어를 안내한다.
