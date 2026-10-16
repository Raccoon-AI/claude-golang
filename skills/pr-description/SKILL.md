---
name: pr-description
description: "Use when given one or more pull request links or numbers and asked to write, rewrite or improve the description, or to make the PR easier to review. Works across repos in one call. Examples: \"/pr-description https://github.com/org/repo/pull/483\", \"rewrite the descriptions of these PRs\", \"make this stack reviewable\""
---

# PR Description for Reviewers

Rewrites each PR body into: what it does, behaviour to know, a file-by-file
review order, tests. Input is PR links (any repos) or bare numbers (current
repo). Nothing else needs to be said: everything below is derived from the PR.
A worked example is in `references/example.md`.

## 1. Resolve each PR

- A link `https://github.com/O/R/pull/N` gives `-R O/R` and `N`. A bare
  number means the current repo. Process in the order given, but for PRs of
  one stack go bottom to top (the one whose base is the default branch first).
- Pull the facts:

```sh
gh pr view N -R O/R --json title,body,url,baseRefName,headRefName,headRefOid,commits,files
```

  `commits[].messageHeadline` gives the story. `files[]` gives
  `path`, `additions`, `deletions`: sort by size, and split test files
  (`_test.go`, `*.test.*`, `__tests__/`) from the rest for the line counts.
- Stack context, derived, never asked: if `baseRefName` is not the default
  branch, `gh pr list -R O/R --head <baseRefName> --json number` names the
  base PR. Count layers by following bases down to the default branch and
  heads up with `gh pr list --base <headRefName>`. A ticket id is the
  `[A-Z]+-\d+` in the branch name, if any.
- Read code before writing. Prefer a local checkout of the repo when one is
  the current directory or a sibling; otherwise read through the API:

```sh
gh api repos/O/R/contents/<path>?ref=<headRefOid> --jq .content | base64 -d
```

  Read the largest non-test files and any file the commits name until you can
  say what the PR does and which files hold the logic. The old body is a hint
  to verify against the diff, not a source.

## 2. Write the body

Use this shape. Drop a section that has nothing to say.

```markdown
<the author's own opening line from the old body, if there is one>

Layer X of Y (TICKET). Base: #NNN or `main`. <No migration. | Proto: ...>

**What it does**
- 3 to 5 bullets. Behaviour, not implementation.

<mermaid flowchart, only for a flow of 3+ stages>

**Behaviour to know**
- Only what surprises a reviewer: gates, flags, retries, what fails until
  something external exists (templates, config, approvals).

**How to review** (about N non-test lines, M of tests)
1. Schema first: migrations, models.
2. Core logic in call order: the file that starts the flow, then what it calls.
3. Wiring and config: constructors, start/stop, flags.
4. Data and copy: label maps, templates. Mark as "skim".
5. Tests last: name the one or two that prove the flow end to end.

<Deleted or generated files that need no reading.>

**Not in this PR.** Only when commits or the old body promise follow-ups.

**Tests.** One line listing what is covered.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

<tool footer from the old body, if any, e.g. gh-stack>
```

## 3. Apply

```sh
gh pr edit N -R O/R --body-file <scratch file>
```

Never change the title. Report one line per PR: link and the new line counts.

## Rules

- Bullets, one idea each, about 20 words. No paragraphs.
- Name files by path from the source root (for Go, under `internal/`) so a
  reviewer can search for them. Group small helpers into one step.
- Line counts come from the PR's file list, split test vs non-test.
- Never cite private planning files or chat; the body must stand on its own.
- Keep the attribution line last, before any tool footer.
- Skip a PR that is merged or closed and say so.
