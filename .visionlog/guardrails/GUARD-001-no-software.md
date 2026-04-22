---
id: "GUARD-001"
type: "guardrail"
title: "No software — skills and templates only"
status: "active"
date: "2026-04-22"
---

## Rule

cli-forge must never become a Python package, npm package, or any installable
CLI. It is a collection of Claude Code skills and file templates. Nothing
more. The CLIs that cli-forge generates are installable software — cli-forge
itself is not.

## Why

Skills are knowledge. They evolve with the agent and never break. The moment
cli-forge becomes a package, it becomes another thing that needs to pass its
own standards, ship its own releases, and carry its own dependency graph —
and the meta-joke writes itself: a tool for reducing agent overhead that
itself adds agent overhead.

Templates are pure knowledge too. They use `{{VARIABLE}}` substitution that
any agent can perform — no template engine, no runtime, no installation.

## Violation Examples

- Adding a `pyproject.toml` with a build system at the cli-forge root
- Creating a `setup.py`, `setup.cfg`, or `package.json` with dependencies
- Publishing cli-forge to PyPI or npm
- Writing Python or TypeScript scripts that need to be installed or executed
  inside cli-forge (scripts that generate a CLI live inside the skill
  instructions, not as separate executables)
- Introducing a template engine (Jinja2, Handlebars, EJS). `{{VAR}}`
  substitution by direct string replacement is the standard.

## Enforcement

`/forge-audit` checks for `pyproject.toml`, `setup.py`, `package.json`, and
any build artifacts in the cli-forge root. Any of these fails the audit.
