# agent-docs

Central documentation for the agent framework ecosystem.

This repo is the single source of truth for all project documentation.
Code repos contain short READMEs pointing here.

## Repositories

| Repo | What it owns |
|---|---|
| [`agent`](https://github.com/ncecere/agent) | Core Go framework and CLI runtime |
| [`agent-plugins`](https://github.com/ncecere/agent-plugins) | Plugin package bundles |
| [`agent-registry`](https://github.com/ncecere/agent-registry) | Plugin registry HTTP server |
| [`agent-docs`](https://github.com/ncecere/agent-docs) | All documentation (this repo) |

---

## Core Framework (`core/`)

The agent runtime, CLI, plugin system, and configuration.

### Concepts

- [plugins.md](core/plugins.md) — Plugin system overview
- [profiles.md](core/profiles.md) — Profile configuration
- [prompts.md](core/prompts.md) — Prompt system
- [policy.md](core/policy.md) — Policy and approval model
- [mcp-bridge.md](core/mcp-bridge.md) — MCP client bridge

### Building Plugins

- [building-plugins.md](core/building-plugins.md) — Plugin authoring guide
- [plugin-runtime-choices.md](core/plugin-runtime-choices.md) — Choosing a runtime (http / mcp / command / host)
- [plugin-http-example.md](core/plugin-http-example.md) — HTTP plugin walkthrough
- [plugin-author-checklist.md](core/plugin-author-checklist.md) — Pre-publish checklist

### Examples

- [build-a-send-email-plugin.md](core/examples/build-a-send-email-plugin.md)
- [build-a-web-research-plugin.md](core/examples/build-a-web-research-plugin.md)
- [build-an-mcp-plugin.md](core/examples/build-an-mcp-plugin.md)

### Patterns

- [patterns/README.md](core/patterns/README.md) — Pattern library index
- [01 Single Tool Agent](core/patterns/01-single-tool-agent.md)
- [02 Research Agent](core/patterns/02-research-agent.md)
- [03 Research and Action Pipeline](core/patterns/03-research-and-action-pipeline.md)
- [04 Orchestrator / Sub-Agents](core/patterns/04-orchestrator-sub-agents.md)
- [05 Policy and Approval Patterns](core/patterns/05-policy-and-approval-patterns.md)
- [06 Prompt Composition](core/patterns/06-prompt-composition.md)

### Architecture Plans

- [go-agent-framework-plan.md](core/architecture/plans/go-agent-framework-plan.md)
- [go-agent-framework-roadmap.md](core/architecture/plans/go-agent-framework-roadmap.md)
- [go-agent-framework-plugin-spec.md](core/architecture/plans/go-agent-framework-plugin-spec.md)
- [go-agent-framework-profiles-and-plugins.md](core/architecture/plans/go-agent-framework-profiles-and-plugins.md)
- [go-agent-framework-package-layout.md](core/architecture/plans/go-agent-framework-package-layout.md)
- [go-agent-framework-cli-surface.md](core/architecture/plans/go-agent-framework-cli-surface.md)
- [go-agent-framework-comparison-and-direction.md](core/architecture/plans/go-agent-framework-comparison-and-direction.md)
- [go-agent-framework-v0.1-feature-list.md](core/architecture/plans/go-agent-framework-v0.1-feature-list.md)
- [go-agent-plugin-package-model.md](core/architecture/plans/go-agent-plugin-package-model.md)

### Release

- [release-checklist-v0.1.md](core/release-checklist-v0.1.md)

---

## Plugin Registry (`registry/`)

The HTTP registry server that serves plugin packages remotely.

- [plugin-registry-contract.md](registry/plugin-registry-contract.md) — HTTP API contract
- [plugin-registry-server-plan.md](registry/plugin-registry-server-plan.md) — Implementation plan
- [registry-server-build-guide.md](registry/registry-server-build-guide.md) — Build and run guide

---

## Plugin Packages (`plugins/`)

The plugin bundle ecosystem and package conventions.

- [overview.md](plugins/overview.md) — Plugin packages overview and layout

---

## Quick Start

```bash
# Run the agent
go run ./cmd/agent --profile ./profiles/my-profile.yaml

# Start the registry server (from agent-registry/)
go run ./cmd/registry-server --plugin-root ../agent-plugins --addr 127.0.0.1:9080

# Search plugins
agent plugins search email
```
