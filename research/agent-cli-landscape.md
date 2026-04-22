# Agent-First CLI Landscape — Raw Research

Collected April 2026. Primary sources for cli-forge design decisions.

## Key Sources

1. **"Writing CLI Tools That AI Agents Actually Want to Use"** — dev.to/uenyioha
   - Why CLI often beats MCP when you have shell access
   - Patterns: `--json`, non-interactive, deterministic, tool discovery via `--help`

2. **"Why CLI Tools Are Beating MCP for AI Agents"** — Jannik Reinhard
   - Context budget / token efficiency as the main reason to prefer CLIs
   - Practical guidelines for CLI-first agents

3. **"Making your CLI agent-friendly"** — Speakeasy
   - Retrofitting existing human CLI to be agent-friendly
   - Remove prompts, spinners, visual noise
   - Add skills/docs that teach agents how to use the CLI
   - Keep human experience but ensure clean machine mode

4. **"Stop Building CLIs for Humans"** — LinkedIn/Chris Rickard
   - Design CLIs assuming the agent is the main user

5. **HN thread on Better-CLI** — skill that suggests making CLIs friendlier for agents

6. **Reddit: "Switched from MCPs to CLIs for Claude Code"** — real practitioners reporting the switch

## Distilled Best Practices

### Interface & Interaction
- Always non-interactive by default — no wizards, no y/N prompts, no menus
- Everything controllable via flags/args/config files
- Deterministic: same inputs → same outputs
- Fast failure with actionable errors ("Missing --project-id; retry with...")

### Output
- `--json` on every command — clean, schema-like output
- `--quiet` mode — bare values, one per line, pipeable
- Token-efficient: paginate by default, require explicit `--limit`, `--filter`
- No banners, tables, colors, spinners in machine mode

### Input
- All required parameters visibly required in `--help`
- Stdin support documented explicitly
- Batch operations (`--all`, `--selector`) over loops

### Discovery (replaces MCP schemas)
- Self-documenting `--help` — concise, structured, skimmable
- Short skill/rule file (50-150 lines) teaching:
  - Canonical commands & workflows
  - When to use `--json`, `--quiet`, stdin
  - Safety constraints
- Minimal but realistic examples (concrete > prose)

## Decision Framework: CLI vs MCP

| Situation | Better fit | Why |
|---|---|---|
| Local coding, tests, linting, debugging | CLI | Lower overhead, faster feedback, shell-native |
| Shared CI/CD, deploy, cross-system workflows | MCP | Centralized auth, structured data, auditability |
| Existing mature terminal tool | CLI + skills | Easier to retrofit than replace |
| Per-user auth, policy boundaries | MCP | Server-level auth, explicit tool boundaries |
| Tool use is ambiguous or complex | Either + skills | Agents need explicit guidance regardless |

## The Thesis

**CLI dominates the inner loop; MCP dominates the governed outer loop; skills/docs are the control layer that makes either usable.**

The key insight: whether the backend is CLI or MCP, the agent still needs a wrapper layer of intent, conventions, examples, and constraints. That wrapper is the skill file.

## Token Economics

```
51-tool MCP:  ~9,000 tokens/session (permanent context cost)
5-command CLI: ~200 tokens/session (skill file, loaded on demand)
Savings:       ~8,800 tokens/session = 98% reduction
```

The MCP tax is paid on every turn. The CLI tax is paid only when the skill is loaded.
