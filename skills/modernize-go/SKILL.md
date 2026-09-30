---
name: modernize-go
description: "Use when the user wants to modernize Go code with `go fix` — apply idiomatic Go 1.27 rewrites (any, min/max, new(expr), slices/maps helpers, range-over-int, strings.Cut, wg.Go, t.Context, etc.), or asks to run go fix / clean up outdated patterns. Examples: \"modernize this code\", \"run go fix\", \"replace interface{} with any\", \"simplify these loops\"."
---

# Modernize Go Code with `go fix`

Go 1.27's `go fix` applies safe, idiomatic rewrites across a package — the
"opportunity for improvement" class of analyzers. Each fix carries a
suggested edit that is safe to apply automatically. This is the preferred way
to modernize; prefer it over hand-editing patterns one by one.

This project is on Go 1.27 (`go.mod`), so all fixers below are available.

## Commands

Preview every change as a unified diff without touching files (do this first):

```sh
go fix -diff ./...
```

Apply the fixes in place:

```sh
go fix ./...
```

Scope to a package or directory instead of the whole module:

```sh
go fix -diff ./internal/backend/service/...
```

Run a single fixer (e.g. only the `new(expr)` rewrite):

```sh
go fix -newexpr ./...
```

Run everything **except** one fixer:

```sh
go fix -stringsbuilder=false ./...
```

## Workflow

1. `go fix -diff ./...` — review the proposed diff. `-diff` exits non-zero when
   there is anything to change, so it doubles as a CI drift check.
2. Apply with `go fix ./...` (or a narrower package path).
3. `go build ./...` then `go test ./...` — the rewrites are mechanical but
   still verify nothing broke.
4. Commit the modernization on its own so the diff is easy to review.

## Available fixers (Go 1.27)

Run `go tool fix help` for the authoritative list and `go tool fix help NAME`
for one fixer's details. As of this toolchain:

| Fixer | What it does |
|-------|--------------|
| `any` | replace `interface{}` with `any` |
| `newexpr` | simplify to Go 1.26's `new(expr)` |
| `minmax` | replace if/else with `min`/`max` |
| `rangeint` | 3-clause `for` → `for range N` |
| `mapsloop` | explicit map loops → `maps` package calls |
| `slicescontains` | loops → `slices.Contains`/`ContainsFunc` |
| `slicessort` | `sort.Slice` → `slices.Sort` for basic types |
| `fmtappendf` | `[]byte(fmt.Sprintf(...))` → `fmt.Appendf` |
| `stringsbuilder` | `+=` accumulation → `strings.Builder` |
| `stringscut` | `strings.Index` patterns → `strings.Cut` |
| `stringscutprefix` | `HasPrefix`/`TrimPrefix` → `CutPrefix` |
| `stringsseq` | ranging over `Split`/`Fields` → `SplitSeq`/`FieldsSeq` |
| `waitgroupgo` | `wg.Add(1)`/`go`/`wg.Done()` → `wg.Go` |
| `testingcontext` | `context.WithCancel` in tests → `t.Context` |
| `reflecttypefor` | `reflect.TypeOf(x)` → `reflect.TypeFor[T]()` |
| `omitzero` | suggest `omitzero` over `omitempty` for struct fields |
| `forvar` | remove redundant loop-variable re-declaration |
| `inline` | apply `//go:fix inline` directive rewrites |
| `errorsastype` | `errors.As` + target var → `errors.AsType[T]` (1.27) |
| `stditerators` | `Len()`/`At(i)` loops → iterator methods (1.27) |
| `slicesbackward` | backward index loops → `slices.Backward` |
| `embedlit` | simplify embedded-field references in composite literals |
| `atomictypes` | basic types in `sync/atomic` calls → atomic types |
| `unsafefuncs` | unsafe pointer arithmetic → function calls |

(`buildtag`, `plusbuild`, `hostport` are also registered.)

## Prerequisite: `./...` and the skill example files

Running `go fix ./...` or `go mod tidy` from the module root fails with
`cannot find module providing package github.com/you/myapp/cmd`. That import
lives in the vendored `cc-skills-golang` example files under
`agent/skills/**/assets/examples/` — documentation snippets, not project code.
The module tooling walks into them because `agent/` isn't a dot-directory.

Fix (idempotent, safe to re-run after any plugin reinstall):

```sh
make isolate-skill-examples   # drops a nested go.mod into each examples dir
```

The nested `go.mod` makes each examples dir its own module, so the parent
module's tooling skips it. `make modernize` runs this first, then applies
`go fix` to the real packages (`./cmd/... ./internal/... ./docs/...`). Prefer
those scoped paths over `./...` regardless.

## Notes

- `go fix` (the modernizer set) is distinct from `gofmt -r` and from the old
  API-migration `cmd/fix`. Here it means the Go 1.27 analyzer-driven fixers.
- `omitzero` is a *suggestion*, not always a safe swap — `omitempty` and
  `omitzero` differ for zero-but-present values. Review those diffs manually.
- The related vendored guidance skill is `golang-modernize` (patterns and
  rationale); this skill is the concrete "run it on this repo" workflow.
- After modernizing generated code paths, prefer regenerating instead — see
  `regenerate-proto` and `regenerate-easyjson`.
