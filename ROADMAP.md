# Roadmap — agent-docs

## Current state

All project documentation lives here, organized under `core/`, `registry/`, and `plugins/`. The docs cover the agent framework, plugin authoring, registry server, and design patterns.

---

## Near term

### Keep docs in sync with code

As features land in `agent`, `agent-registry`, and `agent-plugins`, update the relevant docs here in the same PR or immediately after. The highest priority gaps right now:

- `core/plugins.md` — document remote search, install, upgrade, and publish workflows added in v0.2.0
- `core/profiles.md` — document `profiles install` command added in v0.2.0
- New `core/session-compaction.md` — document the session compaction feature
- New `core/envMapping.md` — document plugin config to env var mapping

### Per-plugin docs in `plugins/`

Each plugin in `agent-plugins` deserves its own page:

```
plugins/
  core-tools.md
  ddg-research.md
  grafana-alerts.md
  grafana-mcp.md
  github-cli.md
  kubectl.md
  slack.md
  send-email.md
  web-research.md
  ...
```

Each page should cover prerequisites, config schema, available tools, and example profile snippets.

---

## Medium term

### CLI reference page
A complete CLI command reference: every command, subcommand, and flag. Auto-generated from source or maintained manually.

### Profile cookbook
A collection of ready-to-use profiles for common tasks:
- research-and-email
- ops-summary (Grafana + Slack)
- coding assistant
- orchestrator with parallel sub-agents

### Architecture diagram
A visual diagram of how the four repos relate to each other at runtime:
- `agent` CLI → `agent-registry` for search/install
- `agent` CLI → plugin runtimes (http, mcp, command, host)
- `agent-registry` ← `agent-plugins` as package source
- `agent-plugins` → `agent` for local installs

---

## Long term

### Versioned docs
When the framework reaches v1.0, version the docs so users on older releases can find accurate information.

### Searchable doc site
Deploy the docs as a static site (e.g. with [mdBook](https://rust-lang.github.io/mdBook/) or a similar tool) with full-text search. The current GitHub-browsable Markdown works but doesn't scale well.
