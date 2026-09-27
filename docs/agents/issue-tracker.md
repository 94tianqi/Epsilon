# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues on **`94tianqi/Epsilon`**. Use the `gh` CLI for all operations.

> **Repo pinning (important).** This clone has two GitHub remotes — `origin` = `94tianqi/Epsilon` (where work is pushed and PRs are opened) and `upstream` = `NekoyaHouse/Epsilon`. Left to infer, `gh` resolves against the upstream, so **every `gh` command below must pass `--repo 94tianqi/Epsilon`**. If you later track issues upstream instead, change the repo in this file.

## Conventions

- **Create an issue**: `gh issue create --repo 94tianqi/Epsilon --title "..." --body "..."`. Use a heredoc for multi-line bodies.
- **Read an issue**: `gh issue view <number> --repo 94tianqi/Epsilon --comments`, filtering comments by `jq` and also fetching labels.
- **List issues**: `gh issue list --repo 94tianqi/Epsilon --state open --json number,title,body,labels,comments --jq '[.[] | {number, title, body, labels: [.labels[].name], comments: [.comments[].body]}]'` with appropriate `--label` and `--state` filters.
- **Comment on an issue**: `gh issue comment <number> --repo 94tianqi/Epsilon --body "..."`
- **Apply / remove labels**: `gh issue edit <number> --repo 94tianqi/Epsilon --add-label "..."` / `--remove-label "..."`
- **Close**: `gh issue close <number> --repo 94tianqi/Epsilon --comment "..."`

## Pull requests as a triage surface

**PRs as a request surface: no.** _(Set to `yes` if this repo treats external PRs as feature requests; `/triage` reads this flag.)_

When set to `yes`, PRs run through the same labels and states as issues, using the `gh pr` equivalents (each with `--repo 94tianqi/Epsilon`): `gh pr view`/`gh pr diff` to read, `gh pr list --state open --json number,title,body,labels,author,authorAssociation,comments` (keep only `authorAssociation` of `CONTRIBUTOR`/`FIRST_TIME_CONTRIBUTOR`/`NONE`), and `gh pr comment`/`gh pr edit --add-label`/`gh pr close`. One number space is shared across issues and PRs, so resolve a bare `#42` with `gh pr view 42` then fall back to `gh issue view 42`.

## When a skill says "publish to the issue tracker"

Create a GitHub issue on `94tianqi/Epsilon`.

## When a skill says "fetch the relevant ticket"

Run `gh issue view <number> --repo 94tianqi/Epsilon --comments`.

## Wayfinding operations

Used by `/wayfinder`. The **map** is a single issue with **child** issues as tickets.

- **Map**: a single issue labelled `wayfinder:map`, holding the Notes / Decisions-so-far / Fog body. `gh issue create --repo 94tianqi/Epsilon --label wayfinder:map`.
- **Child ticket**: an issue linked to the map as a GitHub sub-issue (`gh api` on the sub-issues endpoint). Where sub-issues aren't enabled, add the child to a task list in the map body and put `Part of #<map>` at the top of the child body. Labels: `wayfinder:<type>` (`research`/`prototype`/`grilling`/`task`). Once claimed, the ticket is assigned to the driving dev.
- **Blocking**: GitHub's **native issue dependencies**. Add an edge with `gh api --method POST repos/94tianqi/Epsilon/issues/<child>/dependencies/blocked_by -F issue_id=<blocker-db-id>`, where `<blocker-db-id>` is the blocker's numeric **database id** (`gh api repos/94tianqi/Epsilon/issues/<n> --jq .id`, _not_ the `#number` or `node_id`). GitHub reports `issue_dependencies_summary.blocked_by` (open blockers only). Where dependencies aren't available, fall back to a `Blocked by: #<n>` line at the top of the child body. A ticket is unblocked when every blocker is closed.
- **Frontier query**: list the map's open children (`gh issue list --repo 94tianqi/Epsilon --state open`, scoped to the map's sub-issues / task list), drop any with an open blocker or an assignee; first in map order wins.
- **Claim**: `gh issue edit <n> --repo 94tianqi/Epsilon --add-assignee @me`, the session's first write.
- **Resolve**: `gh issue comment <n> --repo 94tianqi/Epsilon --body "<answer>"`, then `gh issue close <n> --repo 94tianqi/Epsilon`, then append a context pointer to the map's Decisions-so-far.
