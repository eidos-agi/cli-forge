---
name: cli-forge-audit
description: Analyze an MCP server to measure its token tax, identify which tools are actually used, and pick extraction candidates for conversion to a CLI.
user_invocable: true
---

# cli-forge-audit — Measure the MCP Token Tax

Before you convert an MCP into a CLI, you need to know what you're converting and why. This skill audits an MCP server: counts tools, estimates its permanent token cost, separates load-bearing tools from dead weight, and outputs a short-list of extraction candidates.

## Trigger

User says `/cli-forge-audit <path-to-mcp-server>` or asks to audit, analyze, measure, or evaluate an MCP server for CLI extraction.

### Arguments

- `<path>` — path to an MCP server repo, a `.mcp.json` config entry, or a running MCP name. If omitted, run the audit against every MCP in the current project's `.mcp.json`.

## Instructions

### Step 1: Locate the Tool Surface (MANDATORY)

Find every tool the MCP exposes. In order of preference:

1. **Static source** — search the repo for the handler registry. Common patterns:
   - Python FastMCP: `@mcp.tool()` decorators, `server.tool(...)` calls
   - TypeScript MCP SDK: `server.setRequestHandler(...)`, `tools: [...]` exports
   - JSON manifests: `tools.json`, `manifest.json`
2. **Live introspection** — if the MCP is running, call its `tools/list` endpoint (or read the cached schema from `~/.claude/mcp-cache/` when available).
3. **Agent transcript scan** — if neither is available, grep recent session transcripts for `mcp__<server-name>__` invocations.

Output the raw list of tool names before doing anything else. If you cannot find tools, STOP and tell the user where you looked.

### Step 2: Estimate the Token Cost

For each tool, sum the bytes of: name + description + input schema + output schema. Convert to tokens using the 4-chars-per-token rule of thumb (rough but correct to within 20%). Report the total as:

```
<N> tools, ~<K> tokens of permanent context
```

Flag any MCP over 2,000 tokens as "high tax" — it's stealing working memory on every turn.

### Step 3: Usage Scan (MANDATORY)

An MCP with 51 tools where the agent uses 5 is wasting 90% of the tax. Find which tools are actually used:

1. Grep the last 30 days of session transcripts under `~/.claude/projects/**/memory/` and any devlogs for `mcp__<server>__<tool>` references.
2. Count invocations per tool.
3. Sort descending. Report the top 10 and the total count of tools never invoked.

If transcript data is unavailable, ask the user: "Which 3-5 tools from this MCP do you actually use?" Do not guess.

### Step 4: Classify Tools

For each tool, assign a class:

| Class | Meaning | Action |
|-------|---------|--------|
| **CORE** | Used >5 times in 30 days | Convert to CLI command |
| **RARE** | Used 1-5 times | Consider converting; otherwise drop |
| **DEAD** | Never invoked | Drop — do not convert |
| **GOVERNED** | Requires server-side auth, rate limits, or auditability | Keep as MCP — CLI is not appropriate |

GOVERNED tools stay in the MCP. Everything else is a CLI candidate. If every tool is GOVERNED, the MCP is doing its job — do not convert.

### Step 5: Identify the API Client

CLI conversion only works if the tool handlers ultimately call a network API or a library. Locate:

- HTTP client module (requests, httpx, fetch, axios)
- SDK imports (e.g. `from openai import OpenAI`, `import { Octokit } from "@octokit/rest"`)
- Auth pattern (bearer token from env, OAuth flow, mTLS)

If the MCP does pure in-process work with no external calls, it is already the right abstraction — do not convert.

### Step 6: Verdict

Recommend one of:

- **CONVERT** — high tax, clear API client, mostly CORE/RARE tools. Proceed to `/cli-forge-convert`.
- **TRIM** — high tax but only 2-3 tools are CORE. Recommend forking the MCP and deleting the dead tools rather than converting.
- **KEEP** — low tax, or mostly GOVERNED tools. Leave it as an MCP.
- **DELETE** — 0 usage in 30 days. Remove from `.mcp.json` entirely.

## Output Format

```
## CLI-Forge Audit: <mcp-name>

### Token Tax
- Tools exposed: <N>
- Estimated context cost: ~<K> tokens/session
- Tax rating: <LOW | MEDIUM | HIGH>

### Usage (last 30 days)
| Tool | Invocations | Class |
|------|-------------|-------|
| foo_list | 42 | CORE |
| foo_get  | 18 | CORE |
| foo_create | 2 | RARE |
| foo_delete | 0 | DEAD |
| ... | ... | ... |

Unused tools: <N> of <total>

### API Client
- HTTP library: <requests | httpx | fetch | ...>
- Auth: <env var | OAuth | ...>
- Base URL: <...>

### Verdict: <CONVERT | TRIM | KEEP | DELETE>

### Extraction Candidates (if CONVERT)
1. <tool_name> — <one-line purpose>
2. <tool_name> — <one-line purpose>
...

### Projected Savings
Before: ~<K_before> tokens/session
After:  ~<K_after> tokens/session (CLI skill file only)
Savings: ~<delta> tokens/session (<pct>%)

### Next Step
Run `/cli-forge-convert <mcp-name>` with the extraction candidates above.
```

## Rules

- **Read-only.** The audit never modifies source files or `.mcp.json`. Reporting only.
- **Usage data beats intuition.** If transcripts show 0 invocations, the tool is DEAD regardless of how useful the user says it is — ask them to re-run it once to prove it's alive, or drop it.
- **GOVERNED tools stay.** Server-side auth, rate limits, and cross-user policy cannot be replicated by a local CLI. Do not convert them.
- **Never claim savings without measuring.** The "before" number must come from real tool-schema bytes, not a guess.
- **Stop if the MCP has no external API.** Pure in-process MCPs are the right abstraction already.
