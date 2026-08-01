# Rebase Worktrees

Rebase all non-main git worktrees onto a target branch, run package installs, and synchronize .env keys.

## Arguments

- `target branch` (optional): Branch to rebase onto. If omitted, use main worktree's current branch.

## Execution Steps

1. **Collect Worktree Info**
   - Run `git worktree list --porcelain`.
   - Identify main worktree (first entry). Exclude main worktree from rebase targets.
   - If no other worktrees, inform user and stop.

2. **Determine Target Branch**
   - Use provided branch or main worktree's current local branch. Do not fetch.

3. **Rebase Each Worktree**
   - Run `git -C <worktree_path> rebase <target_branch>`.
   - On conflict: run `git rebase --abort`, record failure, proceed to next.
   - On success: record success, proceed to next.

4. **Package Install**
   - For successful worktrees, detect lockfile (`bun.lockb`, `pnpm-lock.yaml`, `yarn.lock`, `package-lock.json`) and run corresponding install command (`bun install`, `pnpm install`, etc.).

5. **Sync .env Keys**
   - If main worktree has `.env`, compare keys with worktrees.
   - If missing/added keys exist, prompt user before updating.

6. **Report Results**
   - Summarize target branch, rebase results (successful/failed), package install status, and `.env` sync status.
