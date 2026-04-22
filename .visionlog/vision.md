---
title: "cli-forge — Kill the MCP Token Tax"
type: "vision"
date: "2026-04-22"
---

## North Star

cli-forge exists for one reason: **turn MCP servers that burn permanent context
into on-demand CLIs that cost nothing until invoked.**

An MCP with 51 tools installs ~9,000 tokens of schema into every session,
forever. Most agents use 5 of those 51 tools. The other 46 are a tax on every
turn — stolen from conversation, code, and reasoning. cli-forge converts the
load-bearing handful into an agent-first CLI and teaches the agent, via a
~200-token skill file, how to use it.

## What cli-forge Contends Against

- **MCP-as-default.** The reflex to ship every integration as an MCP,
  regardless of whether the tool belongs in permanent context.
- **Human-first CLIs.** Tools designed around TTYs, prompts, colors, and
  wizards — unusable by agents without heroic workarounds.
- **Schema sprawl.** 50-tool MCPs where half the tools are vestigial.
  cli-forge's audit step names and shames the dead weight.
- **Invisible CLIs.** A CLI with no skill file is a CLI an agent will
  never discover. Skill generation is non-optional.

## What cli-forge Actually Cares About

In priority order:

### 1. Token economics (the point)
- Measure the real MCP tax in bytes before converting anything
- Report projected savings in tokens/session, verified
- Never ship a CLI + skill that is more expensive than the MCP it replaces

### 2. Agent-First CLI Contract (non-negotiable)
- `--json` on every command
- Non-interactive — no prompts, no wizards, no spinners
- Deterministic — same inputs, same outputs
- Fast failure with actionable errors
- `--dry-run` on destructive operations
- `--help` is the schema

### 3. Skill-first teaching (the bridge)
- Every generated CLI ships with a skill file (~150 lines)
- Examples beat prose — agents pattern-match on real workflows
- Skill file is under 1,000 tokens for a 5-command CLI

### 4. Sharp scope (discipline)
- Convert, don't reinvent. Copy the MCP's API calls verbatim.
- Three skills total: audit, convert, skill. No scope creep.
- Two languages: Python and TypeScript. Everything else is later.

## What cli-forge IS

- Three skills an agent runs to audit an MCP, generate the CLI, and write
  the skill file that teaches the new CLI
- Two CLI scaffolds (Python + TypeScript) that are agent-first by construction
- A skill-template for the generated CLI's own skill file
- The Agent-First CLI Contract, encoded as guardrails

## What cli-forge is NOT

- Not a CLI framework — it generates code that uses existing frameworks
  (typer, commander). We do not invent a new one.
- Not a publisher — ship-forge and foss-forge handle distribution.
- Not a replacement for every MCP. Governed MCPs (server-side auth, rate
  limits, cross-user policy) stay MCPs. The audit step enforces this.
- Not an MCP builder — mcp-forge covers the inverse direction.

## Success Metric

For every MCP that ships into Eidos AGI, an agent can run `/cli-forge-audit`
and get a measured verdict. For every MCP with CONVERT verdict, an agent
can run `/cli-forge-convert` + `/cli-forge-skill` and land a working CLI
with a skill file, remove the MCP from `.mcp.json`, and reduce permanent
context cost by the projected number of tokens — verified, not guessed.
