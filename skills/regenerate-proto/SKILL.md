---
name: regenerate-proto
description: "Use when protobuf/.proto files change and the generated Go code in internal/pb needs regenerating, or when the user asks to regenerate proto / Connect-Go code. Examples: \"regenerate proto\", \"the .proto files changed\", \"rebuild internal/pb\""
---

# Regenerate Protobuf / Connect-Go Code

The generated Go code under `internal/pb` is produced from the `.proto` schema by
`buf`. Never hand-edit files in `internal/pb` — regenerate instead.

## Command

```sh
make generate_proto
```

This runs (see `Makefile`):

```sh
rm -rf internal/pb
buf generate
```

## When to run

- After editing or pulling changes to any `.proto` file (the `proto_schema`
  submodule).
- When `internal/pb` is stale relative to the schema, or a build references a
  proto message/field that doesn't exist yet.

## Notes

- Requires `buf` to be installed and on `PATH`.
- The target deletes `internal/pb` first, so the regeneration is from scratch —
  expect the whole package to be rewritten.
- After regenerating, rebuild (`go build ./...`) to confirm the generated code
  matches its call sites.
