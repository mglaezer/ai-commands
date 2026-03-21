# cm-cleanup

Close the current worktree workspace and remove the worktree.

## Steps

1. Verify we're in a worktree, not the main repo (compare `git rev-parse --show-toplevel` with the main repo root derived from `git rev-parse --git-common-dir`). If in main repo, stop.
2. Check for uncommitted changes. If dirty, list them and ask user to confirm.
3. Record the current workspace ref (`cmux identify`), worktree path, and main repo root.
4. Find another workspace via `cmux list-workspaces`. If none exists, create one at the main repo root: `cmux new-workspace --cwd <main-repo>`.
5. **Critical: run worktree removal, workspace switch, and workspace close in a single Bash call.** Claude Code resets CWD to the project root after each Bash call. Once the worktree is removed, the CWD becomes invalid and no further Bash calls will work. Use `git -C <main-repo-root>` to avoid needing to cd. Example:
   ```
   git -C <main-repo-root> worktree remove --force <worktree-path> && cmux select-workspace --workspace <other-ref> && cmux close-workspace --workspace <self-ref>
   ```
   The cmux commands use a Unix socket and don't need a valid CWD. Session ends after this.

## DANGER — never do these

- Never delete the branch unless the user explicitly asks in this specific request.
- Never force-remove a worktree with uncommitted changes without user confirmation.
- Never close the only workspace without creating a replacement first.
