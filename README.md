# Agent Platform Documentation

A distributed AI agent platform built with Go. Agents are defined by profiles,
powered by plugins, and orchestrated across k8s worker pods via a gateway.

## Platform components

| Component | Repo | Image | Version |
|---|---|---|---|
| **Agent** | [agent](https://github.com/bitop-dev/agent) | `ghcr.io/bitop-dev/agent` | v0.4.5 |
| **Gateway** | [agent-gateway](https://github.com/bitop-dev/agent-gateway) | `ghcr.io/bitop-dev/agent-gateway` | v0.4.4 |
| **Registry** | [agent-registry](https://github.com/bitop-dev/agent-registry) | `ghcr.io/bitop-dev/agent-registry` | v0.2.4 |
| **Plugins** | [agent-plugins](https://github.com/bitop-dev/agent-plugins) | — | 14 packages |
| **Profiles** | [agent-profiles](https://github.com/bitop-dev/agent-profiles) | — | 5+ profiles |
| **Docs** | [agent-docs](https://github.com/bitop-dev/agent-docs) | — | This repo |

## Architecture

```
External clients (API, webhooks, dashboard, CLI, MCP)
                        │
                 ┌──────┴──────┐
                 │   Gateway   │  auth, routing, retries, scheduling,
                 │   :8080     │  webhooks, dashboard, cost tracking
                 └──────┬──────┘
                        │
           ┌────────────┼────────────┐
           │            │            │
     ┌─────┴────┐ ┌────┴─────┐ ┌────┴─────┐
     │ Worker 1  │ │ Worker 2  │ │ Worker N  │  blank pods — pull profiles
     │ :9898     │ │ :9898     │ │ :9898     │  and plugins on demand
     └─────┬────┘ └────┬─────┘ └────┬─────┘
           │            │            │
           └────────────┼────────────┘
                        │
           ┌────────────┼────────────┐
           │            │            │
     ┌─────┴────┐ ┌────┴─────┐ ┌────┴─────┐
     │ Registry  │ │ Postgres  │ │   NATS   │
     │ :9080     │ │ :5432     │ │ :4222    │
     └──────────┘ └──────────┘ └──────────┘
```

Workers start blank. When a task arrives, the worker pulls the profile from
the registry, installs needed plugins on demand, and executes the task.
The gateway handles routing, auth, retries, and stores results in PostgreSQL.

## Documentation index

### Core framework
- [Profiles](core/profiles.md) — agent definitions, discovery metadata, inheritance, MCP server, reactive triggers
- [Plugins](core/plugins.md) — install, upgrade, publish, dependencies, envMapping, env config
- [Policy](core/policy.md) — approval gates, sensitive actions, overlay rules
- [Prompts](core/prompts.md) — system instructions, plugin prompt IDs
- [Building plugins](core/building-plugins.md) — how to create plugins, all contribution types

### Agent patterns
- [1. Single-tool agent](core/patterns/01-single-tool-agent.md) — wrap a CLI/script
- [2. Research agent](core/patterns/02-research-agent.md) — DDG search + fetch
- [3. Research + action pipeline](core/patterns/03-research-and-action-pipeline.md) — research → email
- [4. Orchestrator + sub-agents](core/patterns/04-orchestrator-sub-agents.md) — discovery, delegation, parallel, pipelines
- [5. Policy and approval](core/patterns/05-policy-and-approval-patterns.md)
- [6. Prompt composition](core/patterns/06-prompt-composition.md)

### Gateway
- [Overview](gateway/overview.md) — tasks, auth, webhooks, scheduling, costs, memory, dashboard

### Registry
- [Contract](registry/plugin-registry-contract.md) — API spec

### Key features

| Feature | Description |
|---|---|
| **On-demand everything** | Workers pull profiles + plugins from registry when needed |
| **Model fallback** | `provider.fallback: [model1, model2]` — auto-tries next model on failure |
| **Profile inheritance** | `extends: base-researcher` — merge tools, instructions, config |
| **Agent memory** | `agent/remember` + `agent/recall` — persistent knowledge across tasks |
| **Cost tracking** | Token usage tracked, pricing from models.dev (1800+ models) |
| **Plugin config from env** | `envVar` and `default` in schema — no manual config needed |
| **Reactive triggers** | Service-mode profiles auto-create webhooks for event→task automation |
| **Gateway parallel** | `POST /v1/tasks/parallel` — distribute across k8s pods |
| **Retries + eviction** | Auto-retry on failure, dead workers evicted immediately |
| **MCP server** | `agent serve --profile X` — use agents from opencode/Claude Desktop |
| **Native providers** | OpenAI-compatible + native Anthropic Messages API |
| **Web dashboard** | Embedded at gateway `/` — workers, tasks, agents, costs |

## Quick start

```bash
# Local CLI
agent run --profile researcher "Research AI news"

# HTTP worker
agent serve --addr :9898

# MCP server for external clients
agent serve --profile researcher

# Submit via gateway
curl -X POST http://gateway:8080/v1/tasks \
  -H "Authorization: Bearer <key>" \
  -d '{"profile":"researcher","task":"Top AI story today"}'

# Parallel across workers
curl -X POST http://gateway:8080/v1/tasks/parallel \
  -d '{"tasks":[{"profile":"researcher","task":"Anthropic news"},{"profile":"researcher","task":"OpenAI news"}]}'
```

## k8s deployment

7 pods: gateway + 3 workers + registry + postgres + nats

Workers start blank, auto-register with gateway using pod IPs, and pull
profiles/plugins on demand. Scale workers with `kubectl scale`.

```bash
kubectl -n agent-system scale deployment/agent-workers --replicas=10
```
