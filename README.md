# claude-golang

Shared Claude Code configuration for Raccoon AI Go backends. Mounted as the `.claude/` submodule in consuming repos.

## Contents

- `settings.json` — enables the `cc-skills-golang@samber` plugin (Go best-practice skills)
- `skills/gitnexus/` — GitNexus code-intelligence skills (exploring, impact analysis, debugging, refactoring, PR review)
- `skills/modernize-go/` — apply idiomatic Go rewrites via `go fix`
- `skills/regenerate-easyjson/` — regenerate easyjson marshalers for DB models
- `skills/regenerate-proto/` — regenerate protobuf / Connect-Go code

`settings.local.json` and plugin-installed `skills/golang-*` symlinks are machine-local and gitignored.

## Usage in a repo

```bash
git submodule add git@github.com:Raccoon-AI/claude-golang.git .claude
```

After cloning a repo that uses this submodule:

```bash
git submodule update --init
```
