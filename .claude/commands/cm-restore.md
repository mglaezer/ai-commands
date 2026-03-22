# cm-restore

Restore cmux workspaces for all existing git worktrees that don't already have a workspace open.

## Steps

1. Verify inside a git repo. Resolve main repo root via `git rev-parse --git-common-dir`.
2. `git worktree list` to find all worktrees (skip the main repo entry).
3. `cmux list-workspaces` to find which workspaces already exist.
4. For each worktree that has no matching workspace:
   - `cmux new-workspace --cwd <worktree-path> --command "claude --dangerously-skip-permissions"` (returns ref)
   - `cmux rename-workspace --workspace <ref> <branch>`
   - Set sidebar status (reverse order for correct display):
     - If a PR exists for this branch (`gh pr list --head <branch> --json number --jq '.[0].number'`): `cmux set-status --workspace <ref> pr "#<number>" --icon arrow.triangle.pull`
     - `cmux set-status --workspace <ref> branch "<branch>" --icon arrow.triangle.branch`
     - `cmux set-status --workspace <ref> repo "<repo-name>" --icon folder`
5. Report what was restored.
6. Play a sound when done: `afplay -v 0.10 /System/Library/Sounds/Tink.aiff &`

Note: Use `cmux new-workspace --cwd <path>` instead of `cmux <path>` — the latter steals app focus. Do NOT switch to any restored workspace — just create them. The user can switch manually.
