---
name: cli-forge-convert
description: Generate an agent-first CLI from an MCP server's tool definitions and API client, scaffolding a command tree that follows the Agent-First CLI Contract.
user_invocable: true
---

# cli-forge-convert — MCP to Agent-First CLI

Take the extraction candidates from `/cli-forge-audit` and generate the CLI source. Output is a working scaffold: command tree, flag conventions, API client wiring, tests. Agent-first by construction — no prompts, no wizards, no spinners, `--json` everywhere.

## Trigger

User says `/cli-forge-convert <mcp-name>` or asks to convert, extract, or generate a CLI from an MCP server. Optionally preceded by `/cli-forge-audit`.

### Arguments

- `<mcp-name>` — name of the MCP to convert. Required.
- `--language <python|typescript>` — target language. Defaults to the MCP's own language.
- `--out <dir>` — output directory. Defaults to `./<mcp-name>-cli/`.
- `--tools <comma-list>` — explicit subset of tool names. If omitted, use the CORE and RARE tools from the audit.

## Instructions

### Step 0: Require an Audit (MANDATORY)

Before converting, confirm `/cli-forge-audit` was run against this MCP. If not, run it now. You need:

- List of CORE/RARE tools
- API client details (HTTP library, auth pattern, base URL)
- Verdict of CONVERT (not TRIM/KEEP/DELETE)

If the audit says TRIM/KEEP/DELETE, STOP. Do not convert.

### Step 1: Pick the Template

Choose the scaffold from `templates/`:

| Language | Template | CLI framework |
|----------|----------|---------------|
| Python   | `cli-scaffold.py` | typer (fallback: click) |
| TypeScript | `cli-scaffold.ts` | commander |

If the MCP is in a language not covered, STOP — file an issue before coding from scratch.

### Step 2: Derive the Command Tree

Map each MCP tool to a CLI command. Rules:

1. **Verb-noun naming** — `list_projects` → `projects list`, `create_task` → `tasks create`. Group by resource.
2. **Flatten single-resource MCPs** — if there is only one resource (e.g. a weather MCP), skip the subcommand layer: `weather get --city ...`.
3. **Keep MCP argument names** — don't rename. Agents learn the schema from the MCP; reuse those names as `--flag` equivalents.
4. **Required vs optional** — if the MCP schema says required, make it a required flag with no default.
5. **Enum inputs become Choice types** — typer `Enum` / commander `.choices(...)`.

### Step 3: Wire the API Client

Extract the HTTP calls from the MCP handlers. Do not re-implement — copy the working request code (URL, headers, auth) into a single `client.py` / `client.ts` module. The CLI commands call the client; the client makes the HTTP request.

Auth rule: **environment variables only**. Never prompt, never read from a config file interactively. If the API needs a token, read from `<MCP_NAME>_TOKEN` or the env var the MCP was already using.

### Step 4: Apply the Agent-First CLI Contract (NON-NEGOTIABLE)

Every command the scaffold emits MUST satisfy:

| Rule | Enforcement |
|------|-------------|
| `--json` | Global flag. When set, output is a single JSON object/array on stdout, no other noise. Default ON when `stdout` is not a TTY. |
| Non-interactive | No `input()`, no `prompt()`, no TTY menus. Missing required args → fast error, never a prompt. |
| Deterministic | Same inputs → same outputs. No timestamps, UUIDs, or random content in output unless the command is explicitly generating them. |
| Fast failure | Missing flag → exit 2 with `Error: --<flag> is required. Example: mycli foo --bar baz`. No stack traces by default. |
| `--dry-run` | Required on any command that writes, deletes, or charges money. Prints the request that would be made and exits 0. |
| `--help` is the schema | Each command's help must list every flag, type, default, and one realistic example. This is what replaces the MCP tool description. |
| `--quiet` | One bare value per line, pipeable. For list commands, one ID per line. No headers. |

If the generated code cannot satisfy any of these, STOP and fix the scaffold. Never ship a CLI that prompts.

### Step 5: Generate Tests

Minimum test coverage per command:

- `<command> --help` exits 0 and includes every declared flag
- `<command> --json` with required args returns parseable JSON
- `<command>` with a missing required flag exits 2 and prints "Error: --<flag> is required"
- `<command> --dry-run` (where applicable) does not invoke the real API

Use pytest for Python, vitest or node:test for TypeScript. Mock the API client — no live calls.

### Step 6: Non-Goals

Do NOT add to the CLI:

- Spinners, progress bars, color codes
- Interactive wizards, `questionary`, `inquirer`
- Config-file loaders that prompt for setup
- Shell completion (later, not in the scaffold)
- Windows-specific path handling unless the MCP supported it

Keep the scaffold small. A 200-line CLI beats a 2,000-line one for agents.

## Output Format

```
## CLI-Forge Convert: <mcp-name> -> <cli-name>

### Generated
- <out-dir>/
  - <cli_name>/__init__.py
  - <cli_name>/client.py
  - <cli_name>/commands/<resource>.py
  - tests/test_<resource>.py
  - README.md
  - pyproject.toml (or package.json)

### Commands
| MCP tool | CLI command |
|----------|-------------|
| list_projects | projects list |
| get_project | projects get --id ... |
| ... | ... |

### Contract Verification
| Rule | Status |
|------|--------|
| --json on every command | PASS |
| Non-interactive | PASS |
| Deterministic | PASS |
| Fast failure | PASS |
| --dry-run on destructive ops | PASS |
| --help is the schema | PASS |
| --quiet for piping | PASS |

### Next Step
Run `/cli-forge-skill <cli-name>` to generate the agent skill file,
then run the CLI's own tests and remove the MCP from `.mcp.json`.
```

## Rules

- **No reinvention.** Copy the MCP's existing API calls verbatim; do not redesign auth or retry logic.
- **Contract is non-negotiable.** If the scaffold cannot satisfy every rule in Step 4, the conversion FAILS. Fix the scaffold, do not ship partial compliance.
- **Env vars only for auth.** Never generate a CLI that prompts for credentials.
- **Tests ship with the scaffold.** A CLI without the four basic tests is not shippable.
- **Keep it small.** If the scaffold balloons past 500 lines for a 5-command CLI, something is wrong — usually extra framework ceremony. Trim.
