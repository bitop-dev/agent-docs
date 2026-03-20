# AGENTS.md — agent-docs

Instructions for AI coding agents working in this repository.

## What this repo is

The single source of truth for all project documentation. This is a **docs-only repo** — no code, no build steps, no tests. Every file is Markdown.

## What this repo does NOT own

| Concern | Where it lives |
|---|---|
| Agent framework source code | `../agent` |
| Plugin package bundles | `../agent-plugins` |
| Registry server source code | `../agent-registry` |

Do not add Go source, YAML plugin manifests, or binary artifacts here. Docs only.

## Directory structure

```
README.md               — root index with links to all sections
CHANGELOG.md            — doc changes by version
ROADMAP.md              — what's missing, what to add next
AGENTS.md               — this file

core/                   — agent framework documentation
  plugins.md            — plugin system overview
  profiles.md           — profile configuration reference
  prompts.md            — prompt system
  policy.md             — policy and approval model
  mcp-bridge.md         — MCP client bridge
  building-plugins.md   — plugin authoring guide
  plugin-runtime-choices.md
  plugin-http-example.md
  plugin-author-checklist.md
  release-checklist-v0.1.md
  examples/             — step-by-step plugin build examples
  patterns/             — agent design patterns (6 patterns)
  architecture/plans/   — design and planning documents

registry/               — registry server documentation
  plugin-registry-contract.md   — HTTP API contract
  plugin-registry-server-plan.md
  registry-server-build-guide.md

plugins/                — plugin package documentation
  overview.md           — plugin ecosystem overview
```

## How to contribute docs

1. Find the right section (`core/`, `registry/`, or `plugins/`)
2. Edit or create the Markdown file
3. Update `README.md` if adding a new file that should appear in the index
4. Update `CHANGELOG.md` with what was added or changed
5. Commit and push — no build step needed

## Writing style

- Use plain, direct language
- Prefer short sentences and code examples over long prose
- Every guide should have a working code or YAML example
- Headers: use `##` for major sections, `###` for subsections
- Code blocks: always specify the language (` ```go `, ` ```yaml `, ` ```bash `)
- Link to sibling repos by their GitHub URL, not relative paths (they're separate repos)

## When to update docs here vs in the code repo

**Put it here** if it's:
- User-facing guides (how to build a plugin, how to use profiles)
- Architecture and design documents
- Examples and patterns
- Reference material (CLI commands, config schema)

**Keep it in the code repo** if it's:
- `README.md` — short project summary and quick start only
- `CHANGELOG.md` — release history for that repo's code
- `ROADMAP.md` — next steps for that repo's code
- `AGENTS.md` — agent instructions specific to working in that codebase

## Keeping docs accurate

When code changes in `agent`, `agent-plugins`, or `agent-registry`, check whether any doc here needs updating. The most common gaps:

- New CLI commands or flags not reflected in `core/plugins.md` or `core/profiles.md`
- New plugins in `agent-plugins` not listed in `plugins/overview.md`
- New registry endpoints not in `registry/plugin-registry-contract.md`

The ROADMAP.md in this repo tracks known documentation gaps.
