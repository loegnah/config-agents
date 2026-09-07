---
name: sdlc-lgnh
description: "AI-native software development lifecycle (SDLC) workflow runner."
---

# SDLC-LGNH Skill Runner (AI-Native SDLC)

Execute AI-Native SDLC artifact workflows (`intent.md` -> `spec.md` -> `plan.md`) ONLY when explicitly invoked with `/sdlc-lgnh <keyword>`.

## Keyword Subcommand Matcher

When the user inputs `/sdlc-lgnh <keyword>`, match `<keyword>` against the keywords below to execute the corresponding catalog workflow:

**If invoked without a keyword (`/sdlc-lgnh` only)**: Print the table below (available keywords and workflows), prompt the user to choose a workflow, and exit. Do not read or execute any catalog files.

| Keywords                                                             | Target Workflow             | Catalog File          |
| :------------------------------------------------------------------- | :-------------------------- | :-------------------- |
| `intent`, `draft-intent`, `init`                                     | Stage 1: Intent Capture     | `catalog/intent.md`   |
| `spec`, `draft-spec`, `design`                                       | Stage 2: Spec Specification | `catalog/spec.md`     |
| `plan`, `draft-plan`, `build-plan`                                   | Stage 3: Plan Mode          | `catalog/plan.md`     |
| `run`, `execute`, `exec`, `build`, `impl`, `verify`, `test`, `check` | Run & Verify Plan           | `catalog/run.md`      |
| `simplify`                                                           | Simplify Code               | `catalog/simplify.md` |

## Execution Procedure

1. **Locate Skill Directory**:
   - Find the resolved installation directory of this `skills/sdlc-lgnh/SKILL.md` file.
2. **Read Catalog Instruction**:
   - Read `<sdlc_lgnh_skill_dir>/catalog/<filename>.md` matching the `<keyword>` using `read`.
3. **Execute Steps**:
   - Follow the instructions in the catalog file sequentially.

## No-Keyword Behavior

If `<keyword>` is empty (`/sdlc-lgnh` only), do not execute any catalog. Print the Keyword Subcommand Matcher table above to guide available commands.
