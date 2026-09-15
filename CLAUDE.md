<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **go-backend** (9255 symbols, 51924 relationships, 783 execution flows).

> Index stale? Run `node .gitnexus/run.cjs analyze --index-only` from the project root — it auto-selects an available runner. No `.gitnexus/run.cjs` yet? Bootstrap with `npx`, `bunx`, or `pnpm dlx` — e.g. `bunx gitnexus@latest analyze` (npm 11 npx crash; #1939).

## Always Do

- **MUST run impact before editing.** Use `impact({target: "symbolName", direction: "upstream"})` or `node .gitnexus/run.cjs impact "symbolName" --direction upstream --repo .`; report callers, processes, and risk. Never substitute grep for graph analysis.
- **MUST analyze graph changes before committing.** Use `detect_changes({scope: "all"})` (MCP) or `node .gitnexus/run.cjs detect-changes --scope all --repo .` (CLI fallback). `partial: true` or `truncated: true` is not a clean check — a zero means unseen, not unaffected; re-run it. For regression review: `detect_changes({scope: "compare", base_ref: "main"})` or `node .gitnexus/run.cjs detect-changes --scope compare --base-ref "main" --repo .`.
- MUST warn on HIGH/CRITICAL `risk` pre-edit; never use `riskSharedAxes` to waive a HIGH/CRITICAL `risk` warning. Compare File/symbol: MCP File omits axes; Graph-RAG expands File.
- **MUST treat `risk: UNKNOWN` as unresolved, not as low.** An empty caller set is not evidence the symbol is unused — it can also mean the callers are not resolvable by the index (plain-object property access, dynamic dispatch, cross-language calls). `impact` pairs `UNKNOWN` with a `riskNote` saying so. Confirm with a text search before treating the symbol as safe to change or delete; do not proceed on the strength of a zero.
- **MUST use `query({search_query: "concept"})` for concepts/flows, `context({name: "symbolName"})` for a named symbol, or `impact` for blast radius, on read-only callers, dependencies, imports, or execution flow.** Graph first; text search only for empty/`UNKNOWN`/literals.
- For security review, `explain({target: "fileOrSymbol"})` lists taint findings (source→sink flows; needs `analyze --pdg`).

## Never Do

- NEVER edit a function, class, or method before MCP/CLI impact analysis.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis, and never read `UNKNOWN` as an all-clear — it means the walk could not answer, which is the one verdict that requires confirming by other means.
- NEVER rename symbols with find-and-replace — use `rename` which understands the call graph.
- NEVER commit before MCP/CLI graph change analysis.

## Resources

| Resource | Use for |
| --- | --- |
| `gitnexus://repo/go-backend/context` | Codebase overview, check index freshness |
| `gitnexus://repo/go-backend/clusters` | All functional areas |
| `gitnexus://repo/go-backend/processes` | All execution flows |
| `gitnexus://repo/go-backend/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
| --- | --- |
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus-cli/SKILL.md` |

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
