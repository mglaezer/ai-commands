# code-review

Review the current branch's changes against the default branch. No cmux dependency — works anywhere.

## Steps

1. Detect the default branch (e.g. `main`, `master`).
2. Check for changes (`git diff <default-branch>...HEAD`). If none, report "no changes to review" and stop.
3. Review the diff and commit log. Summarize what changed, flag potential issues, suggest improvements.
