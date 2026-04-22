---
id: "GOAL-001"
type: "goal"
title: "Three core skills: cli-forge-audit, cli-forge-convert, cli-forge-skill"
status: "in-progress"
date: "2026-04-22"
depends_on: []
unlocks: ["GOAL-002"]
---

Three Claude Code skills that cover the full MCP-to-CLI lifecycle. Each skill
is a markdown file with a frontmatter block, a Trigger section, step-by-step
Instructions, an expected Output Format, and a Rules section.

## The Three Skills

1. **`cli-forge-audit`** — Analyze an MCP server. Count tools, estimate the
   permanent token tax, scan session transcripts for real usage, classify
   each tool as CORE / RARE / DEAD / GOVERNED, and issue a verdict
   (CONVERT / TRIM / KEEP / DELETE).

2. **`cli-forge-convert`** — Generate CLI source from the audit's extraction
   candidates. Maps MCP tools to a CLI command tree, wires the existing API
   client verbatim, applies the Agent-First CLI Contract, and emits tests.

3. **`cli-forge-skill`** — Write the agent skill file (~150 lines, under
   1,000 tokens) that teaches agents to use the new CLI. Harvests real
   `--help` output, fills the `skill-template.md` placeholders, and removes
   the replaced MCP from `.mcp.json`.

## Non-Goals

- No fourth skill for "publish the CLI" — foss-forge and ship-forge own
  distribution.
- No skill that keeps both MCP and CLI running in parallel. The whole
  point is to remove the MCP and reclaim the context budget.

## Done When

Running the three skills in order against a real MCP produces: a working
CLI, a skill file, an updated `.mcp.json`, and a measured reduction in
permanent context cost for that project.
