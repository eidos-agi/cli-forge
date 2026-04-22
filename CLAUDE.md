# CLAUDE.md — cli-forge

> Convert MCP servers into agent-first CLIs. 50 tools burning 9K tokens → 5 commands + a skill file.

## What This Is

cli-forge teaches agents how to take an existing MCP server and extract its API logic into an agent-first CLI. The CLI replaces the MCP for environments where shell access exists (Claude Code, Cursor, CI pipelines).

## Why

MCP tool schemas live in context permanently — 51 tools = ~9K tokens eaten before the conversation starts. An agent-first CLI costs zero tokens until invoked, and the skill file teaching its usage is ~200 tokens.

## The Pattern

1. **Audit** the MCP — which tools are actually used? (Usually 5-10 out of 50)
2. **Extract** the API client logic (HTTP calls, auth) into a standalone module
3. **Wrap** with an agent-first CLI: subcommands, `--json`, non-interactive, deterministic
4. **Ship** with a skill file that teaches agents the 4-5 commands they need
5. **Remove** the MCP from `.mcp.json`

## Agent-First CLI Contract

Every CLI produced by cli-forge follows these rules:
- `--json` on every command (default for agent mode)
- Non-interactive — no prompts, no wizards, no spinners
- Deterministic — same inputs, same outputs
- Fast failure with actionable errors ("Missing --project-id, retry with...")
- `--dry-run` for destructive operations
- `--help` is the schema (replaces MCP tool descriptions)
- Batch operations over loops (`--all`, `--selector`)

## Skills

| Skill | What It Does |
|-------|-------------|
| `/cli-forge audit` | Analyze an MCP — tool count, token cost, usage frequency, extract candidates |
| `/cli-forge convert` | Generate CLI source from MCP tool definitions + API client |
| `/cli-forge skill` | Generate the agent skill file for the new CLI |

## Templates

| Template | What It Is |
|----------|-----------|
| `cli-scaffold.py` | Python CLI template with click/typer, --json, --quiet, --dry-run |
| `cli-scaffold.ts` | TypeScript CLI template with commander, same flags |
| `skill-template.md` | Agent skill file template for the generated CLI |

## Related Forges

- **mcp-forge** — the inverse: build new MCP servers from scratch
- **ship-forge** — CI/CD and distribution for the built CLI
- **foss-forge** — open-source packaging standards
