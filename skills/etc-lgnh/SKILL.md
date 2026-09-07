---
name: etc-lgnh
description: "General developer and code explanation workflow runner."
---

# ETC-LGNH Skill Runner

Execute custom developer and code explanation workflows ONLY when explicitly invoked with `/etc-lgnh <keyword>`.

## Keyword Subcommand Matcher

When the user inputs `/etc-lgnh <keyword>`, match `<keyword>` against the keywords below to execute the corresponding catalog workflow:

**If invoked without a keyword (`/etc-lgnh` only)**: Print the table below (available keywords and workflows), prompt the user to choose a workflow, and exit. Do not read or execute any catalog files.

| Keywords                       | Target Workflow | Catalog File         |
| :----------------------------- | :-------------- | :------------------- |
| `eli5`, `explain-5`, `explain` | ELI5 Explainer  | `catalog/eli5.md`    |
| `show-me`, `show`              | Show Me         | `catalog/show-me.md` |

## Execution Procedure

1. **Locate Skill Directory**:
   - Find the resolved installation directory of this `skills/etc-lgnh/SKILL.md` file.
2. **Read Catalog Instruction**:
   - Read `<etc_lgnh_skill_dir>/catalog/<filename>.md` matching the `<keyword>` using `read`.
3. **Execute Steps**:
   - Follow the instructions in the catalog file sequentially.

## No-Keyword Behavior

If `<keyword>` is empty (`/etc-lgnh` only), do not execute any catalog. Print the Keyword Subcommand Matcher table above to guide available commands.
