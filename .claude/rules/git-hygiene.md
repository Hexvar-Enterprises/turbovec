# RULE: NO BRANCH IS LEFT HANGING. CLOSE OUT EVERY BRANCH YOU OPEN.

**Operator directive, 2026-10-01 (Ben, verbatim):**
> "Every time an agent finishes and a PR is merged, it is your responsibility to
> make sure that that branch gets deleted and merged correctly. Also, you are not
> to leave any branches hanging that have any partial commits. It is your job to
> keep up the Git hygiene. If there is a partial commit, it needs to be reviewed
> and evaluated. If there is a full commit and the agent didn't PR it, then it is
> up to you to review and possibly spin up a new sub-agent."

## Who owns it

The **controller** owns it: the session that dispatched the agent, or the agent
itself when it works alone. A sub-agent reports what branch it left and in what
state. The controller closes that branch out. "The sub-agent didn't finish" is
not a terminal state. It is a branch the controller still owes a decision on.

## When it runs

1. **Right after a PR merges.** Close out that branch in the same turn.
2. **When an agent finishes or dies.** Classify its branch before reporting done.
3. **Before a session ends.** Sweep every branch this session created or pushed.

## Classify, then act

Classify by **evidence**, never by name. `claude/*`, `tmp/*` and `scratch-*` get
no exemption for their names. A branch whose content nobody has looked at is
not "probably junk". It is unreviewed.

| Class | Evidence required | Action |
|---|---|---|
| **Merged** | One of these: the branch tip is an ancestor of the trunk; `git merge-tree --write-tree origin/<trunk> <branch>` yields the trunk's own tree (squash-merged content); or the tip equals the head sha GitHub recorded for a merged PR | Delete it: remote, local and worktree. Pre-approved, no ask needed. |
| **Complete, no PR** | Finished work with commits, tests run, but no PR opened | Review the diff, then open a PR (a **draft** if anything is unverified, with the gaps named). If it needs more work, dispatch a sub-agent to finish it. Never delete it. |
| **Partial / WIP** | Unfinished commits, failing tests, TODO stubs, or no clear end state | Review and evaluate. Finish it (sub-agent), fold it into the PR it belongs to, or ask the operator via `AskUserQuestion` whether to abandon it. Never delete it unreviewed. |
| **Superseded** | Its content landed elsewhere under different shas | Prove it commit by commit (trial merge or patch-id). Proven means **Merged**. Anything not proven means **Partial**. |
| **Unknown** | You cannot tell | Treat it as **Partial**. |

A merged PR does not by itself make its branch **Merged**. If commits were
pushed after the merge, the branch now carries content the PR never had.
Check the branch's current tip, not the PR's state.

## Deleting a merged branch

1. Re-read the branch's tip from the remote (`git ls-remote` or the API) in the
   same turn. Never use a sha from memory or a summary.
2. Refuse if any **open** PR uses the branch as its head or its base.
3. Never delete a long-lived branch (`main`, `master`, `develop`,
   `development`, anything in `ship.toml [workflow] long_lived`).
4. Archive the tip first: `refs/tags/archive/<YYYY-MM-DD>/<branch>`. A deleted
   branch is then one command from restored.
5. Delete the remote branch, the local branch and any worktree on it.
6. Run `git fetch --prune origin` so local `origin/*` refs stop showing it.

Where gitguard is installed, `gitguard merge-pr`, `gitguard finish` and
`gitguard gc` already do steps 2, 3, 5 and 6 with these guards. Use them rather
than raw git.

## When deletion is blocked

Cloud sessions sit behind an egress proxy that refuses ref deletion and tag
pushes (HTTP 403). **Do not route around it.** Record the branch, its sha, its
class and the evidence in the operator's cleanup queue (honeycomb
`docs/handoff/`). Say plainly that the deletion is queued and not done.

## Never

- Delete a branch whose content has not been proven in a trunk or reviewed.
- Delete a branch that has commits in the last 24 hours and might belong to a
  live agent. Check before treating it as abandoned.
- Delete a branch with an open PR, or one that is long-lived.
- Force-push, rebase or amend someone else's branch to "tidy" it.
- End a turn with finished work uncommitted. Committed work on a branch is
  recoverable; an uncommitted tree in an ephemeral container is not.

## Reporting

For every branch you closed out, report its repo, branch, sha, class, the
action taken and the evidence behind it. Say "deleted" only for a deletion
you saw succeed. A queued or blocked one is "queued".
