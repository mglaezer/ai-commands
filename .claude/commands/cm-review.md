# cm-review

Open a branch or PR for code review. Same as `/cm-open` but automatically starts a review.

The argument is freeform — it can be a branch name, PR number, or GitHub PR URL.

## Steps

1. Run all steps from `/cm-open`, but when creating the workspace, pass the review prompt to Claude directly:
   `cmux new-workspace --cwd <worktree-path> --command "cd <worktree-path>/<relative-subdir> && claude --dangerously-skip-permissions /code-review"`
2. Rename and switch to the workspace as usual.
3. If the workspace already existed (step 6 in `/cm-open`): after switching, send `/code-review` to the existing Claude session via `cmux send --workspace <ref> "/code-review"` + `cmux send-key --workspace <ref> Enter`.
4. Play a sound when done: `afplay -v 0.10 /System/Library/Sounds/Tink.aiff &`
