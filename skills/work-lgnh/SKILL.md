---
name: work-lgnh
description: "Work log and progress reporting workflow runner."
---

# WORK-LGNH Skill Runner

Execute work log and reporting workflows when invoked with `/work-lgnh <keyword>` or matching reporting intent.

## Rules

- NEVER run `glob`, `grep`, or directory listings to explore or verify catalog files.
- Read the exact relative file path (`catalog/<filename>.md`) directly.

## Workflow Dispatcher

Match the user's keyword or intent against the workflows below:

| Keywords / Intent                               | Target Workflow      | Catalog File                      |
| :---------------------------------------------- | :------------------- | :-------------------------------- |
| `daily`, `worklog`, `report-daily`, `daily-log` | Daily Worklog Report | `catalog/report-daily-worklog.md` |
| `weekly`, `report-weekly`, `weekly-report`      | Weekly Work Report   | `catalog/report-weekly.md`        |

## Execution Procedure

1. **Match Workflow**: Identify the catalog file from the table above.
2. **Read Catalog Directly**: Read `catalog/<filename>.md` directly using relative path.
3. **Execute Steps**: Follow the instructions in the catalog file sequentially.

## Fallback / No-Keyword Behavior

If invoked without a keyword (`/work-lgnh` only) or if no workflow matches:

- Print the table above to guide available commands and ask the user to choose.
- Do not read any catalog files or inspect directories.
