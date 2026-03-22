# cm-list

Show all git worktrees for the current repo with their workspace and PR status.

## Steps

1. Verify inside a git repo. Resolve main repo root via `git rev-parse --git-common-dir`.
2. `git worktree list` to get all worktrees with their branches.
3. `cmux list-workspaces` to find which have open workspaces.
4. `gh pr list --json headRefName,number,url,state` to find PRs for each branch.
5. Display a table:
   ```
   Branch            Worktree                              Workspace    PR
   main              /Users/.../commands-test              —            —
   feature-auth      /Users/.../commands-test-feature-auth workspace:5  #42
   fix-bug           /Users/.../commands-test-fix-bug      (none)       #43
   ```
   - "workspace:N" if a workspace is open, "(none)" if worktree exists but no workspace.
   - PR number if one exists.
