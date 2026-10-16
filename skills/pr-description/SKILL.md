---
name: pr-description
description: "Use when asked to write or rewrite a pull request description so it is short and easy to review, for one or more PR numbers. Examples: \"rewrite the description of #483\", \"write review guidance for PRs 483 484 491\", \"make this PR easier to review\", \"describe this stack for reviewers\""
---

# PR Description for Reviewers

Rewrites a PR body into four parts: what it does, behaviour to know, how to
review (file order), tests. Takes one or more PR numbers. A worked example is
in `references/example.md`.

## Inputs

- PR numbers, e.g. `483 484 491`.
- Optional `-R owner/repo` when the PR is not in the current repo.

## Steps

1. Gather, per PR:

```sh
gh pr view N --json title,body,baseRefName,headRefName,url
git fetch origin
git log --oneline origin/<base>..origin/<head>
git diff --numstat origin/<base>..origin/<head> | awk '{printf "%5d %s\n",$1+$2,$3}' | sort -rn
git diff --numstat origin/<base>..origin/<head> \
  | awk '$3 ~ /_test\.go$/ {t+=$1+$2} $3 !~ /_test\.go$/ {c+=$1+$2} END {print "tests", t, "non-test", c}'
```

2. Read the commit messages and the biggest non-test files until you can say
   what the PR does and which files carry the logic. The old body is a hint,
   not a source: verify against the diff.
3. Write the body with the template below into a scratch file. Keep any line
   the author wrote at the top of the old body.
4. `gh pr edit N --body-file <file>`. Never change the title unless asked.
5. For a stack, do the PRs bottom to top so each body can name its base.

## Template

```markdown
<author's own opening line, if any>

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
3. Wiring and config: service constructors, platform start/stop, flags.
4. Data and copy: label maps, templates. Mark as "skim".
5. Tests last: name the one or two that prove the flow end to end.

<Deleted or generated files that need no reading.>

**Not in this PR.** Only when commits or the old body promise follow-ups.

**Tests.** One line listing what is covered.

🤖 Generated with [Claude Code](https://claude.com/claude-code)

<tool footer from the old body, if any, e.g. gh-stack>
```

## Rules

- Bullets, one idea each, about 20 words. No paragraphs.
- Name files by path under `internal/` (or the repo's source root) so a
  reviewer can search for them. Group small helpers into one step.
- Line counts come from `--numstat`, split test vs non-test.
- Never cite private planning files or chat; the body must stand on its own.
- Keep the attribution line last, before any tool footer.
