# Roadmap

## Completed

| Feature | Version | Status |
|---|---|---|
| Core framework, plugins, profiles, tools, sessions, policy | v0.1.0 | ✅ |
| On-demand plugin + profile install | v0.3.6 | ✅ |
| HTTP workers (blank, self-bootstrapping) | v0.3.0 | ✅ |
| MCP server mode | v0.3.0 | ✅ |
| Sub-agent orchestration (discover, spawn, parallel, pipeline) | v0.3.0 | ✅ |
| Gateway (routing, auth, webhooks, scheduling, retries) | gw v0.1.0 | ✅ |
| NATS event bus + SSE stream | gw v0.2.0 | ✅ |
| Cost tracking (models.dev, token usage) | gw v0.4.0 | ✅ |
| Agent memory (remember/recall) | v0.4.0 | ✅ |
| Model fallback chain | v0.4.5 | ✅ |
| Profile inheritance | v0.4.0 | ✅ |
| Anthropic native provider | v0.4.0 | ✅ |
| Reactive triggers | v0.4.3 | ✅ |
| OpenAI Responses API with token extraction | v0.4.6 | ✅ |
| Configurable model resolution (config/CLI/env/per-profile) | v0.4.7 | ✅ |
| Dashboard (Svelte + shadcn-svelte, 9 pages) | gw v0.5.0 | ✅ |
| Schedule editing in dashboard | gw v0.5.7 | ✅ |
| Marketplace (search, READMEs, downloads, browsable UI) | reg v0.3.0 | ✅ |
| Plugin cleanup (9 Go binary plugins, all with READMEs) | — | ✅ |
| Profile cleanup (6 model-optional profiles) | — | ✅ |
| CI/CD + Docker images + k8s deployment | — | ✅ |

## Remaining

### Multi-user support
- User accounts and authentication
- Per-user config, API keys, and model preferences
- Team/org scoping for plugins and profiles

### Marketplace v2
- Publisher accounts (GitHub OAuth)
- Ratings and reviews
- Download badges and trending
- Package signing/verification

### Production hardening
- Multi-arch plugin builds (CI for linux/amd64 + darwin/arm64)
- Rate limiting on public registry
- Plugin size limits
- Registry replication
