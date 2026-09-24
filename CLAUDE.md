# CLAUDE.md — turbovec

This file holds agent operating rules for the turbovec repository.

---

# RULE: A LOCAL TRUNK NEVER DIVERGES FROM ORIGIN. NON-NEGOTIABLE.

**Operator directive, 2026-09-24.** Written because this recurs across the fleet,
in repo after repo, and is caught late every time.

## The rule

A trunk branch — `main`, `master`, `development`, `develop`, whatever this repo's
default branch actually is — is a **read-only mirror of origin** in every local
checkout. Its only legal local movement is a fast-forward to `origin/<trunk>`.

Without exception:

- **Never commit on a trunk.** Not a docs tweak, not a version bump, not a
  one-line typo fix. Branch first, every time.
- **Never merge, rebase, reset, cherry-pick, revert or amend on a trunk.** The
  one permitted local operation is `git merge --ff-only origin/<trunk>`.
- **Never `git pull` on a trunk.** A bare `pull` is `fetch` + `merge`, and that
  merge creates a local merge commit the moment origin has moved — divergence
  manufactured by the very command meant to prevent it. Use `git fetch origin
  <trunk>` then `git merge --ff-only origin/<trunk>`.
- **Never force-push a trunk.** Not `--force`, not `--force-with-lease`.

## Refreshing is part of branching, not a separate chore

```sh
git fetch origin --prune
git merge --ff-only origin/<trunk>    # run from the trunk; fails loudly if diverged
git switch -c <your-branch>
```

A bare `git pull` while standing on a feature branch does **not** update the
trunk — it merges origin's tip of the branch you are already on. Cutting a branch
from a stale trunk is how one person's divergence becomes everyone's.

## When it has already diverged

Divergence is a **stop condition**, not something to tidy up on the way past.

1. **STOP.** Do not merge, rebase, reset or force-push to "fix" it.
2. **Measure it**: `git rev-list --left-right --count <trunk>...origin/<trunk>`.
3. **Identify the local-only commits**: `git log origin/<trunk>..<trunk>` — and
   whether they exist anywhere else.
4. **Preserve them before anything else**:
   `git branch recovered/<trunk>-$(date +%Y%m%d) <trunk>`. It costs nothing and
   is the difference between a recoverable mistake and lost work.
5. **Report the counts to the operator.** Resolving a diverged trunk is an
   operator decision, never an agent's.

`git reset --hard origin/<trunk>` on a diverged trunk silently destroys every
local-only commit. That is the most expensive way this goes wrong, and it is
exactly why step 4 comes before any remedy.

## Why this is absolute

A diverged trunk is not a local inconvenience. Every branch cut from it inherits
the divergence; every PR from those branches carries phantom commits a reviewer
has to mentally subtract; and the merge that finally reconciles it rewrites
history other checkouts already built on. The cost is paid by everyone, later,
and usually by whoever has the least context.

The asymmetry is the whole argument: `--ff-only` fails loudly and costs one
command to recover from. A merge commit on a trunk is silent, and costs a
reconciliation nobody scheduled.
