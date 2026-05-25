---
id: "GOAL-002"
type: "goal"
title: "Two-language template library: Python + TypeScript scaffolds"
status: "in-progress"
date: "2026-04-22"
depends_on: ["GOAL-001"]
unlocks: []
---

Parameterized CLI scaffolds that `/cli-forge-convert` uses as starting points,
plus a skill-file template that `/cli-forge-skill` fills in. All templates use
simple `{{VARIABLE}}` substitution — no template engine, no runtime.

## The Three Templates

1. **`templates/cli-scaffold.py`** — Python CLI scaffold using `typer`.
   Emits the six Agent-First CLI Contract behaviors out of the box:
   `--json`, `--quiet`, `--dry-run`, non-interactive, deterministic
   output, fast-fail with actionable errors, `--help` as schema.

2. **`templates/cli-scaffold.ts`** — TypeScript equivalent using
   `commander`. Same contract, same flags, same output shape. The
   generated CLI can be published to npm via ship-forge.

3. **`templates/skill-template.md`** — Agent skill file template. Uses
   `{{CLI_NAME}}`, `{{CLI_PURPOSE}}`, `{{COMMANDS}}`, `{{EXAMPLES}}`,
   `{{SAFETY}}`, `{{ENV_VARS}}`, `{{MCP_NAME}}` placeholders. Produces
   a skill file under 1,000 tokens for a 5-command CLI.

## Why Only Two Languages

Python and TypeScript cover >95% of the MCPs currently shipping in the
Eidos AGI ecosystem. Adding Go, Rust, or Ruby templates before any of
those languages appears in a real MCP is scope creep. Revisit when a
real target needs it.

## Placeholder Convention

All placeholders use `{{DOUBLE_BRACE_UPPER_SNAKE}}`. No Jinja `{% %}`
blocks, no Python f-strings, no Handlebars. An agent performing the
substitution does a straight `str.replace()` per variable — zero
dependencies, zero surprises.

## Done When

Running `/cli-forge-convert` against a real MCP produces a working
scaffold from the Python OR TypeScript template, and the generated CLI
passes the Agent-First CLI Contract verification step without manual
edits.
