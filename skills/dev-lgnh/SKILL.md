---
name: dev-lgnh
description: "Developer and software development lifecycle workflow runner."
---

# DEV-LGNH Skill Runner

Execute developer and AI-Native SDLC artifact workflows (`intent.md` -> `spec.md` -> `plan.md` -> `run.md`) when invoked with `/dev-lgnh <keyword>` or matching workflow intent.

## Rules

- NEVER run `glob`, `grep`, or directory listings to explore or verify catalog files.
- Read the exact relative file path (`catalog/<filename>.md`) directly.

## Workflow Dispatcher

Match the user's keyword or intent against the workflows below:

| Keywords / Intent                                                    | Target Workflow             | Catalog File                                     |
| :------------------------------------------------------------------- | :-------------------------- | :----------------------------------------------- |
| `intent`, `draft-intent`, `init`                                     | Stage 1: Intent Capture     | `catalog/sdlc/intent.md`                         |
| `spec`, `draft-spec`, `design`                                       | Stage 2: Spec Specification | `catalog/sdlc/spec.md`                           |
| `plan`, `draft-plan`, `build-plan`                                   | Stage 3: Plan Mode          | `catalog/sdlc/plan.md`                           |
| `run`, `execute`, `exec`, `build`, `impl`, `verify`, `test`, `check` | Run & Verify Plan           | `catalog/sdlc/run.md`                            |
| `simplify`                                                           | Simplify Code               | `catalog/simplify.md`                            |
| `superpowers`                                                        | Superpowers Workflow        | `catalog/superpowers/using-superpowers/SKILL.md` |
| `grill`, `grill-me`, `grilling`                                      | Relentless Interview        | `catalog/matt/grill/SKILL.md`                    |
| `code-review`, `review-diff`, `review`                               | Two-Axis Code Review        | `catalog/matt/code-review/SKILL.md`              |
| `handoff`                                                            | Session Handoff Document    | `catalog/matt/handoff/SKILL.md`                  |

## Execution Procedure

1. **Match Workflow**: Identify the catalog file from the table above.
2. **Read Catalog Directly**: Read `catalog/<filename>.md` directly using relative path.
3. **Execute Steps**: Follow the instructions in the catalog file sequentially.

## Fallback / No-Keyword Behavior

If invoked without a keyword (`/dev-lgnh` only) or if no workflow matches:

- Print the table above to guide available commands and ask the user to choose.
- Do not read any catalog files or inspect directories.
