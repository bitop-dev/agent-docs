# Changelog — agent-docs

All notable changes to the documentation.

---

## Unreleased

---

## v0.2.0

### Added

- `registry/plugin-registry-contract.md` — HTTP API contract for the registry server
- `registry/plugin-registry-server-plan.md` — implementation plan for the registry server
- `registry/registry-server-build-guide.md` — build and run guide for the registry server
- `core/architecture/plans/go-agent-plugin-package-model.md` — plugin package model design
- `core/examples/build-an-mcp-plugin.md` — MCP plugin authoring example
- `core/patterns/` — 6 agent design patterns with worked examples:
  - Single tool agent
  - Research agent
  - Research and action pipeline
  - Orchestrator and sub-agents
  - Policy and approval patterns
  - Prompt composition

### Structure established

- Docs separated from code repos into this dedicated repository
- `core/` — agent framework docs (concepts, guides, examples, patterns, architecture plans)
- `registry/` — registry server docs
- `plugins/` — plugin ecosystem overview

---

## v0.1.0

Initial documentation set.

### Added

- `core/plugins.md` — plugin system overview
- `core/profiles.md` — profile configuration reference
- `core/prompts.md` — prompt system reference
- `core/policy.md` — policy and approval model
- `core/mcp-bridge.md` — MCP client bridge guide
- `core/building-plugins.md` — plugin authoring guide
- `core/plugin-runtime-choices.md` — choosing a runtime
- `core/plugin-http-example.md` — HTTP plugin walkthrough
- `core/plugin-author-checklist.md` — pre-publish checklist
- `core/examples/build-a-send-email-plugin.md`
- `core/examples/build-a-web-research-plugin.md`
- `core/release-checklist-v0.1.md`
- `core/architecture/plans/go-agent-framework-plan.md`
- `core/architecture/plans/go-agent-framework-roadmap.md`
- `core/architecture/plans/go-agent-framework-plugin-spec.md`
- `core/architecture/plans/go-agent-framework-profiles-and-plugins.md`
- `core/architecture/plans/go-agent-framework-package-layout.md`
- `core/architecture/plans/go-agent-framework-cli-surface.md`
- `core/architecture/plans/go-agent-framework-comparison-and-direction.md`
- `core/architecture/plans/go-agent-framework-v0.1-feature-list.md`
- `plugins/overview.md` — plugin package collection overview
