---
name: go-backend-gates
description: "Verification gates and the traps that make a green test run misleading in go-backend. Use before saying a change is done, before committing, and before opening or updating a PR. Also use when adding a guard or validation, when a suite is green but the change feels risky, when touching anything inside a GORM transaction or the cost provider, when ordering a Preload, when comparing errors with errors.Is, and when a stacked PR seems to be getting no Copilot review."
---

# Gates and traps in go-backend

A green suite is evidence about tested code. Several functions here have no
tests at all, so it is routinely evidence about something else.

## Gates

Run in this order. Each has caught a defect the others missed.

| | |
|---|---|
| `go test ./...` | Not the package you edited. Coverage often lives elsewhere — `MoveJobsBetweenRoutes` has no repository test; it is covered by `TestRouteService_MoveJobsBetweenRoutes` in the service package. |
| `rg -l '\bSymbol\b' --type go -g '*_test.go'` | Before trusting green, confirm the thing you changed is tested at all. Nothing back means the suite is silent, not happy. |
| delete each new guard, re-run | A guard whose test still passes without it is not a guard. Restore and re-run green afterwards. |
| `trunk check <files>` | Matches CI. `go vet` alone does not. |

A test that asserts an error passes for the wrong reason whenever something
else on that path raises the same error. Make the fixture valid enough that
only the guard under test can fail, then prove it by deleting the guard.

## Transactions and the cost provider (#374)

The cost provider is a gRPC call to `vrp_solver`. Calling it with a transaction
open holds row locks for the round trip; that is what made a route
uncompletable for four hours in production.

- **`database.GetDB(ctx)` returns the transaction handle inside
  `database.Transactional`.** A repository method called from a service's
  transaction is inside one, however standalone it looks. This is how two live
  defects reached the pricing path.
- `database.IsInTransaction(ctx, db)` sees a transaction on the context or on
  the handle. It cannot see one opened around the call on a different handle.
- `TestNoTransactionalCallerReachesThePricingPath` walks the call graph across
  `internal/` and fails the build if a `Transaction`/`Transactional` closure can
  reach `prefetch`. If it fires, the fix is to give the caller a path that does
  not price, not to silence it.

Price before the transaction opens, then look the plan up inside it.

## Anything computed twice must not come from `time.Now()`

`planTimeForRoute`, `resolveScheduledStart` and `clampReplanAnchor` all answer
`time.Now()` for a route already under way. Computing one in an unlocked pass
and again under the lock gives two values microseconds apart. Where the second
is compared against the first, nothing ever matches, and the operation silently
writes nothing.

Carry the value from the first pass. Do not recompute it.

## `errs.Error` does not unwrap

```go
errors.Is(errs.ErrDatabaseIssue.AddErr(sentinel), sentinel) // false
```

`Error.Is` compares error codes and there is no `Unwrap`, so a sentinel does not
survive wrapping. Check for sentinels **before** wrapping, or the check is dead
code that always reports false.

## Preload order is not guaranteed

A `Preload` with no `Order` returns rows in whatever order the database gives,
and slice position becomes `step_idx`. Order it explicitly.

`step_idx ASC` is not enough: Postgres sorts NULLs last on ASC, SQLite sorts
them first, so tests and production disagree. Use:

```go
Preload("Jobs", func(db *gorm.DB) *gorm.DB {
    return db.Order("step_idx IS NULL, step_idx ASC, id ASC")
})
```

## Route writes go through a state machine

`Route.BeforeUpdate` re-reads the live row and validates the transition against
`routeStateMachine`'s `eventMap`. Saving a struct carrying a stale status fails
as an invalid transition naming a route the caller never touched — confusing,
and it rolls back the whole transaction. Re-read under the lock before writing a
snapshot taken earlier.

`Route.AfterCreate` reaches `audit_log.routeCreated`, which reloads the struct
from the database when `DriverOnShift` is nil. Fields you set before `Create`
can come back changed.

## Staging in an epic worktree

`git add -A` will happily stage a `proto_schema` or `.claude` submodule pointer
that your worktree happens to have checked out older than the branch expects,
reverting the schema on merge. Stage files by name, and read `git diff --cached
--stat` before committing.

## Stacked PRs get no Copilot review

Observed on #492/#493/#494: the PR based on `main` was auto-reviewed within
minutes, the two based on other branches were never reviewed until asked. A
stacked PR is skipped silently, so the layers carrying the riskiest work can
reach a human with no automated review at all. Check rather than assume.

Request it explicitly — REST rejects bot reviewers, so this needs GraphQL:

```bash
PR_ID=$(gh api graphql -f query='{repository(owner:"Raccoon-AI",name:"go-backend"){
  pullRequest(number:NNN){id}}}' --jq '.data.repository.pullRequest.id')
gh api graphql -f query="mutation{requestReviews(input:{
  pullRequestId:\"$PR_ID\",botIds:[\"BOT_kgDOCnlnWA\"],union:true}){pullRequest{number}}}"
```

`gh pr view --json reviewRequests` does not list bots; query `reviewRequests`
through GraphQL to confirm it registered.

Unresolved review threads block merge under the repo ruleset, so reply and
resolve rather than leaving a thread answered-but-open.
