# Issue tracker: GitLab

Issues and specs for this repo live as GitLab issues. Use the [`glab`](https://gitlab.com/gitlab-org/cli) CLI for all operations.

## Conventions

- **Create an issue**: `glab issue create --title "..." --description "..."`. Use a heredoc for multi-line descriptions. Pass `--description -` to open an editor.
- **Read an issue**: `glab issue view <number> --comments`. Use `-F json` for machine-readable output.
- **List issues**: `glab issue list -F json` with appropriate `--label` filters.
- **Comment on an issue**: `glab issue note <number> --message "..."`. GitLab calls comments "notes".
- **Apply / remove labels**: `glab issue update <number> --label "..."` / `--unlabel "..."`. Multiple labels can be comma-separated or by repeating the flag.
- **Close**: `glab issue close <number>`. `glab issue close` does not accept a closing comment, so post the explanation first with `glab issue note <number> --message "..."`, then close.
- **Merge requests**: GitLab calls PRs "merge requests". Use `glab mr create`, `glab mr view`, `glab mr note`, etc., the same shape as `gh pr ...` with `mr` in place of `pr` and `note`/`--message` in place of `comment`/`--body`.

Infer the repo from `git remote -v`; `glab` does this automatically when run inside a clone.

## Merge requests as a triage surface

**MRs as a request surface: no.** _(Set to `yes` if this repo treats external merge requests as feature requests; `/triage` reads this flag.)_

When set to `yes`, MRs run through the same labels and states as issues, using the `glab mr` equivalents:

- **Read an MR**: `glab mr view <number> --comments` and `glab mr diff <number>` for the diff.
- **List external MRs for triage**: `glab mr list -F json`, then keep only MRs whose author is not a project member/owner (a contributor's MR, not a maintainer's in-flight work).
- **Comment / label / close**: `glab mr note`, `glab mr update --label`/`--unlabel`, `glab mr close`.

Unlike GitHub, GitLab numbers issues and MRs separately, so `#42` is unambiguous once you know which surface the maintainer means.

## When a skill says "publish to the issue tracker"

Create a GitLab issue.

## When a skill says "fetch the relevant ticket"

Run `glab issue view <number> --comments`.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue whose tickets are **task**
work items parented to it.

- **Map**: a single **issue** labelled `wayfinder:map`, holding the Notes /
  Decisions-so-far / Fog body. `glab issue create --label wayfinder:map`.
  Epics can hold the map only in a **group** namespace on Premium/Ultimate; in a
  personal namespace `/type Epic` is rejected, so default to an issue.

- **Child ticket**: a **task** work item parented to the map, labelled
  `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, it
  is assigned to the driving dev.

  **An issue cannot parent another issue.** GitLab's `allowedChildTypes` for
  Issue is `Task` only, so tickets must be tasks. The quick actions do not help:
  `/set_parent`, `/add_child` and `/promote_to` fail (silently on some
  instances, `Could not apply` on others) and leave the map showing
  `Child items 0`, with the hierarchy existing only as prose.

  **There is no in-place conversion.** `/type Task` is rejected and
  `WorkItemUpdateInput` has no `workItemTypeId`, so a ticket created as an issue
  must be deleted and recreated as a task.

  Create with REST (which takes description and labels directly), then set the
  parent over GraphQL:

  ```bash
  glab api --method POST "projects/:id/issues" \
    -f title="..." -f issue_type=task -f labels="wayfinder:grilling" \
    -f description="$(cat body.md)"

  glab api graphql -f query='mutation { workItemUpdate(input: {
    id: "gid://gitlab/WorkItem/<task>",
    hierarchyWidget: {parentId: "gid://gitlab/WorkItem/<map>"}
  }) { workItem { iid } errors } }'
  ```

  Resolve global ids with:
  `{ project(fullPath:"<group>/<repo>") { workItems(iids:["<iid>"]) { nodes { id } } } }`

  Tasks keep everything a ticket needs: labels, assignees, native blocking
  links, and they appear in `glab issue list` and `projects/:id/issues` (as
  `issue_type: task`), so the frontier query below is unaffected.

- **Blocking**: GitLab's **native blocking link**, the canonical, UI-visible
  representation. Add it with the `/blocked_by #<n>` quick action, posted as a
  note (`glab issue note <child> --message "/blocked_by #<blocker>"`). Native
  blocking links are a Premium/Ultimate feature; on the free tier fall back to a
  `Blocked by: #<n>, #<n>` line at the top of the description. A ticket is
  unblocked when every blocker is closed.
- **Frontier query**: `glab issue list -F json` scoped to the map's children,
  drop any with an open blocker: a native `blocked_by` link to an open issue
  (`glab api projects/:id/issues/:iid/links`), or an open issue in the
  `Blocked by` line, or an assignee; first in map order wins.
- **Claim**: `glab issue update <n> --assignee @me`, the session's first write.
- **Resolve**: `glab issue note <n> --message "<answer>"`, then
  `glab issue close <n>`, then append a context pointer (gist + link) to the
  map's Decisions-so-far.
