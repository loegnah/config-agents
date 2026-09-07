# Branch Clear Gone

Delete local git branches marked [gone] and their worktrees.

## Execution Steps

1. **List Branches**
   - Run `git branch -v` to identify branches with `[gone]` status.
   - Note: Branches with a '+' prefix have associated worktrees.

2. **Identify Worktrees**
   - Run `git worktree list`.

3. **Remove Worktrees & Delete Branches**
   - Process `[gone]` branches and remove associated worktree if present.
   - Execute:
     ```bash
     git branch -v | grep '\[gone\]' | sed 's/^[+* ]//' | awk '{print $1}' | while read branch; do
       echo "Processing branch: $branch"
       worktree=$(git worktree list | grep "\\[$branch\\]" | awk '{print $1}')
       if [ ! -z "$worktree" ] && [ "$worktree" != "$(git rev-parse --show-toplevel)" ]; then
         echo "  Removing worktree: $worktree"
         git worktree remove --force "$worktree"
       fi
       echo "  Deleting branch: $branch"
       git branch -D "$branch"
     done
     ```

4. **Report Results**
   - Report which worktrees and branches were removed. If none marked as `[gone]`, report no cleanup was needed.
