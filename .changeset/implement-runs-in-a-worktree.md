---
"marius-mattpocock-skills": patch
---

`/implement` now builds in its own git worktree on its own branch by default, cut from the current `HEAD` before the first edit, and installs the project there. It commits to that branch and hands back the branch name, the worktree path, and the `git worktree remove` line, leaving the merge and the cleanup to you. It builds in your checkout only when you ask for that directly. Docs page, README entries, and `ask-matt` re-synced.
