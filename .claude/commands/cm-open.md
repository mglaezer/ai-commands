# cm-open

Open a branch or PR in a worktree + workspace. Switch to it if it already exists.

The argument is freeform — it can be a branch name, PR number, or GitHub PR URL.

## Steps

1. Verify the current directory is inside a git repo. Resolve main repo root via `git rev-parse --git-common-dir`.
2. Parse input:
   - Branch name → use directly.
   - PR number or GitHub PR URL → resolve to branch via `gh pr view`.
3. Check if branch exists (locally, remotely). If not, create it from the default branch.
4. Check if the branch is already checked out anywhere (`git worktree list`). If it's checked out in the main repo, warn the user and stop. If it's in another worktree at an unexpected path, inform and offer to use that path instead.
5. Worktree path: `<parent-of-main-repo>/<repo-name>-<sanitized-branch>`. Sanitize: replace non-alphanumeric chars with `-`.
6. If worktree + workspace already exist: **switch to it** (`cmux select-workspace --workspace <ref>`) and stop.
7. If worktree exists but no workspace: `cmux new-workspace --cwd <worktree-path> --command "claude --dangerously-skip-permissions"` (returns the new workspace ref), `cmux rename-workspace --workspace <new-ref> <branch>`, switch to it.
8. If no worktree: create it (for remote-only branches use `git worktree add -b <branch> <path> origin/<branch>`), `cmux new-workspace --cwd <worktree-path> --command "claude --dangerously-skip-permissions"` (returns the new workspace ref), `cmux rename-workspace --workspace <new-ref> <branch>`, switch to it.

Note: Use `cmux new-workspace --cwd <path>` instead of `cmux <path>` — the latter steals app focus. `cmux` commands require workspace refs (e.g. `workspace:5`), not names. Always capture and use the ref returned.

## DANGER — never do these

- Never `git checkout` or `git switch` in the current directory. Worktrees exist to avoid this.
- Never delete a branch unless the user explicitly asks.
- Never remove or overwrite an existing worktree without asking.
- Never close the workspace you're currently in from this command.
