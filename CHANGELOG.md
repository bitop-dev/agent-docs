# Changelog

## Agent v0.4.5
- Fix: model fallback stream error handling — pop failed model and retry
- Fix: `imagePullPolicy` cache issue with re-tagged images

## Agent v0.4.3
- Agent memory: `agent/remember` and `agent/recall` host tools
- Anthropic provider: extract usage tokens from Messages API response
- Reactive triggers: service-mode profiles register webhooks at startup
- Memory tool descriptors added to spawn-sub-agent plugin

## Agent v0.4.1
- Fix: workers register with pod IPs instead of hostnames
- Plugin config from env: `Property.EnvVar` and `Property.Default` auto-populate

## Agent v0.4.0
- Agent memory host operations (via gateway `/v1/memory`)
- Model fallback chain: `provider.fallback: [model1, model2]`
- Plugin config from environment: `envVar`, `default`, convention
- Profile inheritance: `extends` with merge logic
- Worker auto-registration with gateway via `GATEWAY_URL`
- Native Anthropic Messages API provider
- Reactive trigger foundation: `mode: service`, `triggers` in profile spec

## Agent v0.3.6
- Gateway-distributed parallel: `GATEWAY_URL` dispatches spawn-parallel through gateway
- On-demand profile install from registry
- On-demand plugin install with config validation
- Registry-aware agent discovery (local + remote)
- Sub-agent progress visibility (`[sub:profile]` prefix)
- MCP server keepalive notifications

## Agent v0.3.0
- Agent discovery (`agent/discover`)
- Structured handoff (`context` parameter)
- Agent pipelines (`agent/pipeline` with `{{var}}` routing)
- Pipeline checkpoints
- MCP server mode (`agent serve --profile`)
- HTTP worker mode (`agent serve --addr :9898`)
- Worker-to-worker HTTP dispatch
- Parallel sub-agents (`agent/spawn-parallel`)
- Session compaction (pi-mono style)
- Transitive plugin dependency auto-install
- Version pinning (`name@version`)
- Plugin upgrade and publish
- Tool name sanitization for Bedrock/Azure

## Agent v0.1.0
- Core framework, CLI, built-in tools
- OpenAI-compatible provider
- SQLite sessions, policy, approvals
- Plugin system, MCP client bridge

---

## Gateway v0.4.4
- Fix: dead worker eviction on connection failure
- Fix: prefer original provider pricing for duplicate model IDs in models.dev

## Gateway v0.4.1
- Fix: `agent_memory` and `cost_tracking` tables in migration

## Gateway v0.4.0
- Agent memory API (`/v1/memory`)
- Cost tracking with models.dev pricing sync (1800+ models)
- Admin pricing management (`/v1/costs/pricing`)
- Cost recording from worker token usage reports

## Gateway v0.3.2
- Automatic retry on transient task failures (timeouts, 502s)
- Dead worker eviction

## Gateway v0.3.1
- Parallel task dispatch (`POST /v1/tasks/parallel`)

## Gateway v0.3.0
- Embedded web dashboard at `/`
- NATS event bus integration
- SSE event stream (`/v1/events`)

## Gateway v0.2.0
- NATS event bus, lifecycle events

## Gateway v0.1.0
- Task routing and worker management
- API key auth with scopes
- Webhooks with template expansion
- Cron scheduling
- PostgreSQL state store

---

## Registry v0.2.4
- Profile publish endpoint (`POST /v1/profiles`)
- `--base-url` flag for k8s artifact URLs
- Empty plugin-root graceful handling
- Multi-version package support
- Worker registration endpoints (moved to gateway in v0.4+)
