---
name: work-lgnh
description: "Work log and progress reporting workflow runner."
---

# WORK-LGNH Skill Runner

Execute work log and reporting workflows ONLY when explicitly invoked with `/work-lgnh <keyword>`.

## Keyword Subcommand Matcher

When the user inputs `/work-lgnh <keyword>`, match `<keyword>` against the keywords below to execute the corresponding catalog workflow:

**If invoked without a keyword (`/work-lgnh` only)**: Print the table below (available keywords and workflows), prompt the user to choose a workflow, and exit. Do not read or execute any catalog files.

| Keywords                                        | Target Workflow      | Catalog File                      |
| :---------------------------------------------- | :------------------- | :-------------------------------- |
| `daily`, `worklog`, `report-daily`, `daily-log` | Daily Worklog Report | `catalog/report-daily-worklog.md` |
| `weekly`, `report-weekly`, `weekly-report`      | Weekly Work Report   | `catalog/report-weekly.md`        |

## Execution Procedure

1. **Locate Skill Directory**:
   - Find the resolved installation directory of this `skills/work-lgnh/SKILL.md` file.
2. **Read Catalog Instruction**:
   - Read `<work_lgnh_skill_dir>/catalog/<filename>.md` matching the `<keyword>` using `read`.
3. **Execute Steps**:
   - Follow the instructions in the catalog file sequentially.

## No-Keyword Behavior

If `<keyword>` is empty (`/work-lgnh` only), do not execute any catalog. Print the Keyword Subcommand Matcher table above to guide available commands.
