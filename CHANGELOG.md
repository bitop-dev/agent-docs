# Changelog

## Agent v0.3.6
- Gateway-distributed parallel: `SpawnSubRunParallel` dispatches through gateway when `GATEWAY_URL` is set
- On-demand profile install from registry
- On-demand plugin install with config validation
- Registry-aware agent discovery (local + remote profiles)
- Sub-agent progress visibility (`[sub:profile]` prefix)
- MCP server keepalive notifications
- Better error reporting (show both local and registry errors)

## Agent v0.3.0
- Agent discovery (`agent/discover` tool)
- Structured handoff (`context` parameter on spawn/pipeline)
- Agent pipelines (`agent/pipeline` with `{{var}}` routing)
- Pipeline checkpoints
- MCP server mode (`agent serve --profile`)
- HTTP worker mode (`agent serve --addr :9898`)
- Worker-to-worker HTTP dispatch
- Worker registration with registry
- Message bus for agent-to-agent communication
- Parallel sub-agents (`agent/spawn-parallel`)
- Session compaction (pi-mono style)
- Transitive plugin dependency auto-install
- Version pinning (`name@version` syntax)
- Plugin upgrade and publish commands
- Tool name sanitization for Bedrock/Azure

## Gateway v0.3.2
- Parallel task dispatch: `POST /v1/tasks/parallel` across workers
- Automatic retry on transient failures (timeouts, 502s)
- NATS event bus integration
- SSE event stream endpoint
- Embedded web dashboard

## Gateway v0.1.0
- Task routing and worker management
- API key auth with scopes
- Webhooks with template expansion
- Cron-based scheduling
- PostgreSQL state store

## Registry v0.2.4
- Profile package support (publish, index, serve)
- Worker registration endpoints (moved to gateway in v0.3+)
- `--base-url` flag for k8s artifact URLs
- Empty plugin-root graceful handling
- Multi-version package support
- Publish endpoint with bearer auth

## Agent v0.1.0
- Core framework release
- Built-in tools: read, write, edit, bash, glob, grep
- OpenAI-compatible provider
- SQLite sessions
- Plugin system with install/enable/disable
- Policy and approval gates
- MCP client bridge
