# cli-forge

Convert MCP servers into agent-first CLIs. Kill the token tax.

## The Problem

An MCP with 51 tools costs ~9K tokens of context — permanently. That's context space stolen from conversation, code, and reasoning. And most agents use 5 of those 51 tools.

## The Solution

Extract the API logic into a CLI. Ship with a skill file. Remove the giant MCP.

MCP can remain as a tiny shim when a host needs one, but its job is to point at the CLI, not to model the whole tool universe. The CLI provides progressive reveal of the deeper graph.

```
Before: 51 MCP tools → ~9,000 tokens/session
After:  5 CLI commands + skill → ~200 tokens/session
```

## Skills

```bash
/cli-forge audit     # Analyze an MCP: tool count, token cost, which tools are actually used
/cli-forge convert   # Generate CLI source from MCP API client
/cli-forge skill     # Generate the agent skill file
```

## The Agent-First CLI Contract

- `--json` on every command
- Non-interactive (no prompts, no wizards)
- Deterministic (same input → same output)
- Fast failure with actionable errors
- `--dry-run` for destructive ops
- `--help` is the schema
- `status`, `doctor`, `list`, `find`, and `ask` commands reveal deeper domains without loading every tool up front

## License

MIT
