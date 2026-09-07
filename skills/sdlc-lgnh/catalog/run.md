# Stage 3 Execution: Run & Verify Plan

Execute the approved `plan.md` sequentially, verify against acceptance criteria, and maintain `plan.md` as the single source of truth.

## Language

Respond in the same language the user is using (e.g., Korean if the user communicates in Korean). Match the user's language for status reports and discussions.

## Arguments

- `target` (optional): Path to target story directory or `plan.md` file.

## Constraints

- Strictly adhere to **Order of Work** and **Files that change** in `plan.md`.
- If implementation diverges or unexpected adjustments are needed, update `plan.md` simultaneously (Single Source of Truth).
- Avoid unrequested abstractions or speculative complexity.
- **Never claim success without runtime checks**: Always execute verification criteria from `plan.md`'s `Proof` section. Never relax or weaken tests arbitrarily; fix implementation code when checks fail.

## Execution Steps

1. **Locate & Validate `plan.md`**
   - If argument provided, resolve `<target_dir>`.
   - If omitted:
     - Search `docs/story/` for the most recently modified directory containing `plan.md`.
   - Read `<target_dir>/plan.md`. If missing, instruct user to run `/sdlc-lgnh plan` first.
   - Verify plan status is approved or ready for implementation.

2. **Sequential Implementation**
   - Follow `Order of Work` step-by-step.
   - For each step:
     - Apply surgical, clean code changes using the appropriate editing tools.
     - Implement only files enumerated in `Files that change`.
   - If unforeseen adjustments are required during implementation:
     - Update `plan.md` immediately to reflect modified file list or execution order.

3. **Verify & Self-Correction Loop**
   - Read and execute criteria defined in `plan.md`'s `## 4. 검증 및 완료 증거 (Proof)`:
     - Automated tests (e.g., `bun test`, `pytest`, `npm test`)
     - Linter / typecheck / build commands (e.g., `bun run check`)
   - If any test or lint check fails:
     - Diagnose root cause in implementation files.
     - Fix the bug in application code and re-run verification until all pass cleanly.

4. **Update Plan Status & Report Deliverable Evidence**
   - Once implementation is verified with clean checks, update `plan.md` status line:
     - `상태: [구현완료(completed)]`
   - Output deliverable proof:
     - Test run output showing pass count.
     - Lint/typecheck output with 0 errors.
   - Inform user that the feature is implemented, verified, and ready.
