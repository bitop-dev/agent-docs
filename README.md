# Agent Platform Documentation

A distributed AI agent platform built in Go. Agents are defined by profiles,
powered by plugins, orchestrated across k8s worker pods, and discoverable
through a marketplace.

## Platform components

| Component | Repo | Image | Version |
|---|---|---|---|
| **Agent** | [agent](https://github.com/bitop-dev/agent) | `ghcr.io/bitop-dev/agent` | v0.4.8 |
| **Gateway** | [agent-gateway](https://github.com/bitop-dev/agent-gateway) | `ghcr.io/bitop-dev/agent-gateway` | v0.5.7 |
| **Registry** | [agent-registry](https://github.com/bitop-dev/agent-registry) | `ghcr.io/bitop-dev/agent-registry` | v0.3.0 |
| **Plugins** | [agent-plugins](https://github.com/bitop-dev/agent-plugins) | — | 9 packages |
| **Profiles** | [agent-profiles](https://github.com/bitop-dev/agent-profiles) | — | 6 profiles |
| **Docs** | [agent-docs](https://github.com/bitop-dev/agent-docs) | — | This repo |

## Architecture

```
Clients (API, webhooks, dashboard, CLI, MCP)
                    │
             ┌──────┴──────┐
             │   Gateway   │  auth, routing, retries, scheduling,
             │   :8080     │  webhooks, dashboard, cost tracking
             └──────┬──────┘
                    │
       ┌────────────┼────────────┐
       │            │            │
  ┌────┴────┐ ┌────┴────┐ ┌────┴────┐
  │Worker 1 │ │Worker 2 │ │Worker N │  blank pods — pull profiles
  │ :9898   │ │ :9898   │ │ :9898   │  and plugins on demand
  └────┬────┘ └────┬────┘ └────┬────┘
       │            │            │
       └────────────┼────────────┘
                    │
       ┌────────────┼────────────┐
       │            │            │
  ┌────┴────┐ ┌────┴────┐ ┌────┴────┐
  │Registry │ │Postgres │ │  NATS   │
  │ :9080   │ │ :5432   │ │ :4222   │
  │(market) │ │         │ │         │
  └─────────┘ └─────────┘ └─────────┘
```

## Documentation index

### Core framework
- [Profiles](core/profiles.md) — agent definitions, inheritance, model resolution, MCP server, reactive triggers
- [Plugins](core/plugins.md) — install, upgrade, publish, dependencies, envMapping, env config
- [Building plugins](core/building-plugins.md) — Go binary plugins, tool definitions, README
- [Policy](core/policy.md) — approval gates, sensitive actions, overlay rules
- [Prompts](core/prompts.md) — system instructions, plugin prompt IDs

### Agent patterns
- [1. Single-tool agent](core/patterns/01-single-tool-agent.md)
- [2. Research agent](core/patterns/02-research-agent.md)
- [3. Research + action pipeline](core/patterns/03-research-and-action-pipeline.md)
- [4. Orchestrator + sub-agents](core/patterns/04-orchestrator-sub-agents.md)
- [5. Policy and approval](core/patterns/05-policy-and-approval-patterns.md)
- [6. Prompt composition](core/patterns/06-prompt-composition.md)

### Gateway
- [Overview](gateway/overview.md) — tasks, auth, webhooks, scheduling, costs, memory, dashboard

### Registry & Marketplace
- [Registry contract](registry/plugin-registry-contract.md) — API spec
- [Marketplace](registry/marketplace.md) — search, detail, download counts, README

### Available plugins
- [Plugin catalog](plugins/overview.md) — all 9 plugins with tools and descriptions

### Key features

| Feature | Description |
|---|---|
| **Marketplace** | Browsable web UI for discovering plugins + profiles with search, downloads, READMEs |
| **Model resolution** | Config-level default + per-profile overrides + CLI flag. Profiles are model-optional. |
| **Responses API** | OpenAI Responses API with multi-turn tool calls and token extraction |
| **On-demand everything** | Workers pull profiles + plugins from registry when needed |
| **Model fallback** | `provider.fallback: [model1, model2]` — auto-tries next model on failure |
| **Profile inheritance** | `extends: base-researcher` — merge tools, instructions, config |
| **Agent memory** | `agent/remember` + `agent/recall` — persistent knowledge across tasks |
| **Cost tracking** | Token usage tracked per task, pricing from models.dev (1800+ models) |
| **Plugin config from env** | `envVar` and `default` in schema — no manual config needed |
| **Reactive triggers** | Service-mode profiles auto-create webhooks for event→task automation |
| **Gateway parallel** | `POST /v1/tasks/parallel` — distribute across k8s pods |
| **Dashboard** | Svelte + shadcn-svelte: tasks, workers, agents, plugins, costs, memory, webhooks, schedules |
| **MCP server** | `agent serve --profile X` — use agents from opencode/Claude Desktop |

## Quick start

```bash
# Install
go install github.com/bitop-dev/agent/cmd/agent@latest

# Configure
cat > ~/.agent/config.yaml << EOF
providers:
  openai:
    baseURL: https://api.openai.com/v1
    apiKey: sk-...
    model: gpt-4o
    apiMode: responses
EOF

# Run an agent
agent run --profile researcher "Summarize today's AI news"

# Override model for one run
agent run --profile researcher --model gpt-4o-mini "Quick summary"

# HTTP worker
agent serve --addr :9898

# MCP server for external clients
agent serve --profile researcher
```

## k8s deployment

6 pods: gateway + registry + 2 workers + postgres + nats

```yaml
# config.yaml — model set once for all profiles
providers:
  openai:
    baseURL: https://api.openai.com/v1
    apiKey: sk-...
    model: gpt-4o
    apiMode: responses
    models:                    # optional per-profile overrides
      code-reviewer: gpt-4o   # best model for code review
      writer: gpt-4o-mini     # cheaper for writing tasks
```

Workers start blank, pull profiles/plugins on demand. Scale with kubectl:
```bash
kubectl -n agent-system scale deployment/agent-workers --replicas=10
```
