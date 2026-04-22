---
name: cli-forge-skill
description: Generate the agent skill file that teaches a Claude Code agent how to use a newly converted CLI — the replacement for the MCP tool schemas.
user_invocable: true
---

# cli-forge-skill — Teach the Agent to Use the New CLI

A CLI with no skill file is invisible to agents. After `/cli-forge-convert` produces the source, this skill writes the short markdown file (~150 lines) that teaches agents the canonical commands, when to use `--json`, and where the safety boundaries are. This is what replaces the 9K-token MCP schema.

## Trigger

User says `/cli-forge-skill <cli-name>` or asks to generate the skill file, agent docs, or usage rules for a converted CLI.

### Arguments

- `<cli-name>` — name of the CLI built by `/cli-forge-convert`. Required.
- `--out <path>` — where to write the skill file. Defaults to `.claude/skills/<cli-name>.md` in the current project.
- `--template <path>` — override the default `templates/skill-template.md`.

## Instructions

### Step 0: Require a Converted CLI (MANDATORY)

Find the CLI source produced by `/cli-forge-convert`. You need:

- The full list of commands
- Each command's `--help` output (real, by invoking it — not imagined)
- At least one realistic example per command

If the CLI is not yet built, STOP and run `/cli-forge-convert` first.

### Step 1: Harvest the Real Help Text

For every command, run:

```
<cli-name> <command> --help
```

Capture the output verbatim. This becomes the source of truth for flags, defaults, and types. **Never invent a flag the CLI does not expose.** If `--help` disagrees with the skill file, the skill is wrong — rewrite it.

### Step 2: Fill the Skill Template

Start from `templates/skill-template.md`. Substitute:

| Variable | Source |
|----------|--------|
| `{{CLI_NAME}}` | CLI binary name (e.g. `mycli`) |
| `{{CLI_PURPOSE}}` | One-line purpose (e.g. "Query and mutate MyService projects") |
| `{{COMMANDS}}` | Markdown table: command, purpose, one canonical invocation |
| `{{EXAMPLES}}` | 3-6 concrete agent workflows, each 2-4 lines |
| `{{SAFETY}}` | Destructive commands + the flag needed to actually execute them |
| `{{ENV_VARS}}` | Required environment variables and where to set them |

### Step 3: Write Realistic Examples (MANDATORY)

The examples section is the skill's load-bearing element. Agents pattern-match on examples more than prose. Rules:

1. **Every example uses `--json`.** Agent-mode is JSON-mode.
2. **Every example is a real workflow**, not a flag showcase. Example: "List open projects then fetch the three most recent" — two commands, piped or sequential.
3. **Use `jq` for JSON post-processing.** Agents know `jq`; don't reinvent with Python.
4. **Show `--dry-run` for every destructive workflow.** The skill must teach the safety pattern.
5. **No more than six examples.** If you need more, the CLI is too big — split it.

### Step 4: Token Budget Check

The finished skill must come in under 250 tokens per 100 lines (rule of thumb: ~1000 tokens total for a 5-command CLI skill). If it is longer:

- Collapse repeated flag descriptions into a single "Global flags" table
- Remove prose that duplicates what `<cmd> --help` says
- Cut examples down to the highest-impact three

This is the whole point of the forge. A 2000-token skill file defeats the exercise.

### Step 5: Safety Section

Call out:

- Commands that write or delete: list them with the required flag to actually execute (e.g. `--confirm`)
- Commands that charge money or hit rate-limited APIs: note the quota and the env var that gates them
- Commands that must never run against production without `--dry-run` first

If there are no destructive commands, say so explicitly: "All commands are read-only; no safety gates required."

### Step 6: Install the Skill

Write to `.claude/skills/<cli-name>.md` in the target project. Never overwrite an existing skill with the same name — if one exists, append `-v2` and ask the user to pick which to keep.

Update the project's `.mcp.json` in the same commit: remove the MCP entry the CLI replaces. If the MCP is still needed for GOVERNED tools (see `/cli-forge-audit`), leave the entry but note in the skill that the MCP still exists for those specific tools.

### Step 7: Verify

Before returning, run one realistic example from the skill against the CLI and confirm it works end-to-end. If it fails, the skill is broken — fix it before reporting success.

## Output Format

```
## CLI-Forge Skill: <cli-name>

### Skill File
- Path: <project>/.claude/skills/<cli-name>.md
- Size: <N> lines, ~<K> tokens
- Commands covered: <list>

### MCP Replacement
- Removed from .mcp.json: <mcp-name> (or "left in place — GOVERNED tools remain")
- Token saving: ~<before> -> ~<after> per session

### Verification
- <cmd> --help exits 0: PASS
- Example workflow from skill runs end-to-end: PASS
- Skill under token budget: PASS

### Next Step
Commit the CLI + skill, update the project README to reference the new skill,
and run a session with a fresh agent to confirm it picks up the skill without hints.
```

## Rules

- **`--help` is ground truth.** Never describe a flag the CLI does not expose. When in doubt, re-run `--help` and copy.
- **Examples > prose.** Agents learn workflows from concrete examples. Keep prose tight.
- **Under 1000 tokens for a 5-command CLI.** If it exceeds, cut. The token savings are the product.
- **Never overwrite an existing skill.** Append a suffix and ask the user to reconcile.
- **End-to-end test before shipping.** Run one example from the skill against the CLI. If it fails, the skill is fiction.
- **Delete the MCP in the same commit** once the skill is verified. Leaving both online is the worst of both worlds — the MCP tax persists and agents get confused about which to use.
