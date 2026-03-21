# Roadmap

## Completed

| Feature | Version | Status |
|---|---|---|
| Core framework | v0.1.0 | ✅ |
| Plugin system (install, upgrade, publish, deps) | v0.3.0 | ✅ |
| Profile system (discovery, on-demand, registry) | v0.3.6 | ✅ |
| On-demand plugin + profile install | v0.3.6 | ✅ |
| HTTP workers (dynamic, blank, self-bootstrapping) | v0.3.0 | ✅ |
| MCP server mode | v0.3.0 | ✅ |
| Agent discovery (local + registry) | v0.3.0 | ✅ |
| Structured handoff (context) | v0.3.0 | ✅ |
| Pipelines + checkpoints | v0.3.0 | ✅ |
| Sequential + parallel sub-agents | v0.3.0 | ✅ |
| Gateway (routing, auth, webhooks, scheduling) | gw v0.1.0 | ✅ |
| Gateway parallel dispatch | gw v0.3.1 | ✅ |
| Gateway retries + dead worker eviction | gw v0.4.4 | ✅ |
| NATS event bus + SSE stream | gw v0.2.0 | ✅ |
| Web dashboard | gw v0.3.0 | ✅ |
| Task history (PostgreSQL) | gw v0.1.0 | ✅ |
| Cost tracking (models.dev, 1800+ models) | gw v0.4.0 | ✅ |
| Agent memory (agent/remember, agent/recall) | v0.4.0 | ✅ |
| Model fallback chain | v0.4.5 | ✅ |
| Plugin config from env | v0.4.0 | ✅ |
| Profile inheritance | v0.4.0 | ✅ |
| Anthropic native provider | v0.4.0 | ✅ |
| Reactive triggers (service mode) | v0.4.3 | ✅ |
| Worker auto-registration (pod IPs) | v0.4.1 | ✅ |
| Session compaction (pi-mono style) | v0.3.0 | ✅ |
| Tool name sanitization | v0.3.0 | ✅ |
| CI/CD + Docker images + k8s deployment | v0.3.0 | ✅ |

## Remaining

### Enhanced dashboard
- Task detail view with output, tool call trace, timing
- Submit tasks from the UI
- Real-time SSE event feed (currently polling)
- Cost charts by profile and model
- Plugin and profile management

### Marketplace
- Public registry at registry.bitop.dev
- Community plugin and profile contributions
- Rating, downloads, verified publishers

### Notes
- Cost tracking shows $0 for self-hosted models (UFL proxy doesn't return usage tokens) — revisit when using direct API keys
- Multi-arch plugin builds (CI for linux/amd64 + darwin/arm64) deferred until plugins are rebuilt as official packages
