# Agent Platform Documentation

Documentation for the distributed AI agent platform built with Go.

## Platform components

| Component | Repo | Image | Description |
|---|---|---|---|
| **Agent** | [agent](https://github.com/bitop-dev/agent) | `ghcr.io/bitop-dev/agent` | Framework, CLI, workers |
| **Gateway** | [agent-gateway](https://github.com/bitop-dev/agent-gateway) | `ghcr.io/bitop-dev/agent-gateway` | Task routing, auth, webhooks, scheduling, dashboard |
| **Registry** | [agent-registry](https://github.com/bitop-dev/agent-registry) | `ghcr.io/bitop-dev/agent-registry` | Plugin + profile package server |
| **Plugins** | [agent-plugins](https://github.com/bitop-dev/agent-plugins) | — | 14 plugin packages |
| **Profiles** | [agent-profiles](https://github.com/bitop-dev/agent-profiles) | — | Agent profile definitions |
| **Docs** | [agent-docs](https://github.com/bitop-dev/agent-docs) | — | This repository |

## Architecture

```
External clients (API, webhooks, web dashboard, CLI)
                        │
                 ┌──────┴──────┐
                 │   Gateway   │  auth, routing, retries, scheduling, dashboard
                 └──────┬──────┘
                        │
           ┌────────────┼────────────┐
           │            │            │
     ┌─────┴────┐ ┌────┴─────┐ ┌────┴─────┐
     │ Worker 1  │ │ Worker 2  │ │ Worker N  │  on-demand profiles + plugins
     └─────┬────┘ └────┬─────┘ └────┬─────┘
           │            │            │
           └────────────┼────────────┘
                        │
           ┌────────────┼────────────┐
           │            │            │
     ┌─────┴────┐ ┌────┴─────┐ ┌────┴─────┐
     │ Registry  │ │ Postgres  │ │   NATS   │
     │ packages  │ │  state    │ │  events  │
     └──────────┘ └──────────┘ └──────────┘
```

Workers start blank and pull profiles and plugins from the registry on demand.
The gateway routes tasks to workers, retries on failures, and stores results
in PostgreSQL. NATS provides real-time events for the dashboard.

## Documentation index

### Core framework
- [Profiles](core/profiles.md) — agent definition spec, discovery metadata, MCP server
- [Plugins](core/plugins.md) — install, upgrade, publish, dependencies, envMapping
- [Policy](core/policy.md) — approval gates, sensitive actions, overlay rules
- [Prompts](core/prompts.md) — system instructions, plugin prompt IDs
- [Building plugins](core/building-plugins.md) — how to create plugins from scratch

### Patterns
- [Single-tool agent](core/patterns/01-single-tool-agent.md)
- [Research agent](core/patterns/02-research-agent.md)
- [Research + action pipeline](core/patterns/03-research-and-action-pipeline.md)
- [Orchestrator + sub-agents](core/patterns/04-orchestrator-sub-agents.md) — includes parallel, pipelines, discovery
- [Policy and approval](core/patterns/05-policy-and-approval-patterns.md)
- [Prompt composition](core/patterns/06-prompt-composition.md)

### Gateway
- [Overview](gateway/overview.md) — tasks, auth, webhooks, scheduling, retries, dashboard

### Registry
- [Contract](registry/plugin-registry-contract.md)
- [Server plan](registry/plugin-registry-server-plan.md)

### Deployment
- k8s manifests in `agent-deploy/k8s/` (not in this repo — see operator's deployment)
- Docker images: `ghcr.io/bitop-dev/{agent,agent-gateway,agent-registry}`

## Quick start

```bash
# Local development
go run ./cmd/agent run --profile researcher "Research AI news"

# HTTP worker
agent serve --addr :9898

# MCP server (for opencode/Claude Desktop)
agent serve --profile researcher

# Submit via gateway
curl -X POST http://gateway:8080/v1/tasks \
  -H "Authorization: Bearer <key>" \
  -d '{"profile":"researcher","task":"Top AI story today"}'

# Parallel tasks via gateway
curl -X POST http://gateway:8080/v1/tasks/parallel \
  -d '{"tasks":[{"profile":"researcher","task":"Anthropic news"},{"profile":"researcher","task":"OpenAI news"}]}'
```
