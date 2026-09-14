<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **go-backend** (12852 symbols, 44714 relationships, 300 execution flows). Use the GitNexus MCP tools to understand code, assess impact, and navigate safely.

> If any GitNexus tool warns the index is stale, run `npx gitnexus analyze` in terminal first.

## Always Do

- **MUST run impact analysis before editing any symbol.** Before modifying a function, class, or method, run `gitnexus_impact({target: "symbolName", direction: "upstream"})` and report the blast radius (direct callers, affected processes, risk level) to the user.
- **MUST run `gitnexus_detect_changes()` before committing** to verify your changes only affect expected symbols and execution flows.
- **MUST warn the user** if impact analysis returns HIGH or CRITICAL risk before proceeding with edits.
- When exploring unfamiliar code, use `gitnexus_query({query: "concept"})` to find execution flows instead of grepping. It returns process-grouped results ranked by relevance.
- When you need full context on a specific symbol — callers, callees, which execution flows it participates in — use `gitnexus_context({name: "symbolName"})`.

## Never Do

- NEVER edit a function, class, or method without first running `gitnexus_impact` on it.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis.
- NEVER rename symbols with find-and-replace — use `gitnexus_rename` which understands the call graph.
- NEVER commit changes without running `gitnexus_detect_changes()` to check affected scope.

## Resources

| Resource | Use for |
|----------|---------|
| `gitnexus://repo/go-backend/context` | Codebase overview, check index freshness |
| `gitnexus://repo/go-backend/clusters` | All functional areas |
| `gitnexus://repo/go-backend/processes` | All execution flows |
| `gitnexus://repo/go-backend/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
|------|---------------------|
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

# Documentation Style

Applies to any documentation Claude writes: README files, docs/, ADRs, PR descriptions, code comments longer than a line.

## Always Do

- Keep it short. Say it once, in simple words. Prefer bullet points over paragraphs.
- Use the STAR method (Situation, Task, Action, Result) when describing a change, fix, or decision.
- Use a Mermaid diagram (or similar) when a flow, sequence, or structure is easier to see than to read.
- Use numbered steps with a concrete example when a procedure is easier to show than to explain.

## Never Do

- NEVER pad documentation with restated context, filler, or obvious detail.
- NEVER write a paragraph when a bullet list or table says the same thing.
- NEVER write documentation that a reader has to scroll through to find the one line they need.


# Before Coding

- **(MUST)** Ask clarifying questions for ambiguous requirements.
- **(MUST)** Draft and confirm an approach (API shape, data flow, failure modes) before writing code.
- **(SHOULD)** When >2 approaches exist, list pros/cons and rationale.
- **(SHOULD)** Define testing strategy (unit/integration) and observability signals up front.

# Workflow

- **(SHOULD)** Try to make minimal changes - do not refactor unrelated code.
- **(NEVER)** Modify files outside the scope of the current task without asking.
- **(SHOULD)** Avoid adding large comments into self explanatory functions and code.
- **(MUST)** Be precise and simple when adding comments. Don't over-explain. Must be simple!

# Logging & Observability

- **(MUST)** Structured logging (`sirupsen/logrus`) with levels and consistent fields.
- **(SHOULD)** Correlate logs/metrics/traces via request IDs from context.
- **(SHOULD)** Add info logs to capture logic when tracing issues on production.

