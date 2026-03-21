# Testing Methodology

## Setup

Tests use a proxy workspace pattern:

1. Create a test repo (`commands-test`) with a GitHub remote and at least one PR.
2. Open it as a cmux workspace (`commands-test`).
3. Spawn Claude with `--dangerously-skip-permissions` in that workspace.
4. From an orchestrating workspace, send `/cm-...` commands to `commands-test` via `cmux send` + `cmux send-key`.
5. Read results via `cmux read-screen` after a delay.
6. Verify state with `cmux list-workspaces`, `git worktree list`, and `git branch --list`.

This approach tests the commands exactly as a user would invoke them, without modifying the orchestrating session.

## Key discoveries during testing

- `cmux <path>` steals app focus. Use `cmux new-workspace --cwd <path>` instead.
- `cmux rename-workspace <name>` without `--workspace <ref>` renames the CURRENT workspace, not the intended one.
- `cmux` commands require refs (`workspace:5`), not names.
- Claude Code resets CWD to the project root after each Bash call. Once a worktree directory is removed, all subsequent Bash calls fail. The workaround: run worktree removal, workspace switch, and workspace close in a single `&&`-chained Bash call.
- `git worktree remove` without `--force` refuses to remove worktrees with untracked files, even after user confirmation. Use `--force` after confirmation.
- Polling `cmux read-screen` with `grep '>'` for Claude readiness is unreliable — Claude's prompt `❯` is unicode. Use `cmux new-workspace --command` to pass the initial command directly.
- `claude [prompt]` accepts an initial prompt as a CLI argument, so `--command "claude --dangerously-skip-permissions /code-review"` works without sleep/polling.

## Tests

### Test 1: `/cm-open test-branch` — existing local branch, no worktree
**Precondition:** Branch `test-branch` exists locally, no worktree or workspace for it.
**Expected:** Creates worktree, creates workspace with `cmux new-workspace --cwd --command`, renames workspace, switches to it.
**Verify:** `cmux list-workspaces` shows new workspace, `git worktree list` shows new worktree.

### Test 2: `/cm-open test-branch` — worktree + workspace already exist
**Precondition:** Worktree and workspace for `test-branch` are already open.
**Expected:** Detects existing worktree + workspace, switches to it via `cmux select-workspace`, does NOT recreate.
**Verify:** No new workspace created, focus switched.

### Test 3: `/cm-open 1` — PR number
**Precondition:** PR #1 exists on the repo.
**Expected:** Resolves PR #1 to branch name via `gh pr view`, creates worktree + workspace.
**Verify:** `cmux list-workspaces` shows workspace named after the PR's branch.

### Test 4: `/cm-open <PR URL>` — GitHub PR URL
**Precondition:** PR #1 exists, no worktree for its branch.
**Expected:** Parses URL, resolves to branch, creates worktree + workspace.
**Verify:** Same as test 3.

### Test 5: `/cm-open main` — branch checked out in main repo
**Precondition:** `main` is the currently checked-out branch in the main repo.
**Expected:** Detects `main` is already checked out, warns user, stops. Does NOT create worktree.
**Verify:** No new workspace or worktree created.

### Test 6: `/cm-open pr-test-branch` — worktree exists, no workspace
**Precondition:** Worktree exists on disk but its workspace was closed.
**Expected:** Detects existing worktree, skips `git worktree add`, creates workspace only.
**Verify:** `cmux list-workspaces` shows new workspace, `git worktree list` unchanged.

### Test 7: `/cm-cleanup` — clean worktree
**Precondition:** Inside a worktree workspace with no uncommitted changes.
**Expected:** Finds another workspace, removes worktree + closes workspace in single Bash call.
**Verify:** Workspace gone from `cmux list-workspaces`, worktree gone from `git worktree list`, branch still exists.

### Test 8: `/cm-cleanup` — dirty worktree
**Precondition:** Inside a worktree workspace with an untracked file.
**Expected:** Lists dirty files, asks for confirmation. After confirmation, uses `--force` to remove worktree.
**Verify:** Same as test 7 after confirmation.

### Test 9: `/cm-cleanup` — from main repo
**Precondition:** Running in the main repo workspace (not a worktree).
**Expected:** Detects main repo, refuses to clean up, shows clear message.
**Verify:** No workspaces or worktrees affected.

### Test 10: `/cm-review 1` — fresh workspace
**Precondition:** No worktree or workspace for PR #1's branch.
**Expected:** Creates worktree + workspace with `--command "claude --dangerously-skip-permissions /code-review"`. Claude starts and immediately runs the review.
**Verify:** `cmux read-screen` on new workspace shows code review output.

### Test 11: `/cm-review 1` — workspace already exists
**Precondition:** Worktree and workspace for PR #1's branch are already open with Claude running.
**Expected:** Switches to existing workspace, then sends `/code-review` via `cmux send` + `cmux send-key`.
**Verify:** `cmux read-screen` shows a new code review ran in the existing session.
