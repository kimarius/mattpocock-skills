---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

## Work in a fresh workspace

Every run builds in its own **worktree**, on its own **branch**, created before the first edit. The user's checkout keeps its branch, its index, and its uncommitted work, so a run that goes wrong is thrown away by deleting a directory, and several runs can go at once without fighting over one `HEAD`.

Set it up first:

1. Pick a short slug for the work, from the ticket reference or the spec title: `42-rate-limit`.
2. From the repo, branch off the current `HEAD` into a sibling directory: `git worktree add ../<repo>-<slug> -b implement/<slug>`.
3. Run every read, edit, command, and test inside that directory for the rest of the run.
4. Get the project building there before writing code. A new worktree carries the tracked files only, so install dependencies (`pnpm install`, or whatever this project uses) and copy across the untracked local files the build and the tests need, such as `.env`.

Build in the user's own checkout when they ask for that directly: "work here", "stay on this branch", "use the branch I'm on".

## Build it

Use /tdd where possible, at pre-agreed seams.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, use /code-review to review the work.

Commit your work to the run's branch.

## Hand it back

Leave the worktree on disk and report three things: the branch name, the worktree path, and `git worktree remove <path>` to clean it up. Merging, opening a PR, and removing the worktree are the user's calls.
