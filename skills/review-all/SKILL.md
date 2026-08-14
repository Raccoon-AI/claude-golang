---
name: review-all
description: "Combined review — correctness bugs + over-engineering, compressed output, context-mode for big diffs. Use when the user wants a full review pass over the current diff or a PR: \"review all\", \"review this PR\", \"full review\", \"/review-all\"."
---

Combined review pass over current diff. If $ARGUMENTS names a PR (number or URL), review that PR's diff via `gh pr diff` instead.

Steps:
1. Get the diff (`git diff` vs base branch, or `gh pr diff`). If it's large (>500 lines or many files), do not dump it raw into context — use context-mode (`ctx_fetch_and_index` / `ctx_execute`) to index and query it instead of reading it whole.
2. Invoke the `code-review` skill on the diff for correctness bugs.
3. Invoke the `ponytail:ponytail-review` skill on the same diff for over-engineering (unneeded deps, speculative abstractions, reinvented stdlib, dead flexibility).
4. Merge both finding lists, drop duplicates.
5. Output using caveman-review's compressed format, one line per finding: `path:line: <severity emoji> <severity>: <problem>. <fix>.` No praise, no scope creep, skip pure formatting nits.
6. If a finding is Go-specific (concurrency, nil-safety, error wrapping, etc.), let the relevant `cc-skills-golang` skill inform the fix suggestion.
7. If neither pass finds anything, say so in one line — don't pad.
