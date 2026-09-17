---
name: brain-lgnh
description: "Second brain knowledge capture and query workflow runner."
---

# BRAIN-LGNH Skill Runner

Execute second brain knowledge capture and query workflows (`~/note/brain`) when invoked with `/brain-lgnh <keyword>` or matching brain workflow intent.

## Rules

- NEVER run `glob`, `grep`, or directory listings to explore or verify catalog files.
- Read the exact relative file path (`catalog/<filename>.md`) directly.
- Strictly adhere to second brain constraints: NEVER run `ingest` or `init` workflows.

## Workflow Dispatcher

Match the user's keyword or intent against the workflows below:

| Keywords / Intent                                  | Target Workflow | Catalog File               |
| :------------------------------------------------- | :-------------- | :------------------------- |
| `capture`, `brain-capture`, `save`, `note`, `memo` | Brain Capture   | `catalog/brain-capture.md` |
| `query`, `brain-query`, `search`, `find`           | Brain Query     | `catalog/brain-query.md`   |

## Execution Procedure

1. **Match Workflow**: Identify the catalog file from the table above.
2. **Read Catalog Directly**: Read `catalog/<filename>.md` directly using relative path.
3. **Execute Steps**: Follow the instructions in the catalog file sequentially.

## Fallback / No-Keyword Behavior

If invoked without a keyword (`/brain-lgnh` only) or if no workflow matches:

- Print the table above to guide available commands and ask the user to choose.
- Do not read any catalog files or inspect directories.
