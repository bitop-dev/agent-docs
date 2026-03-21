# Changelog

## Agent v0.4.8
- Fix: Responses API multi-turn tool calls (inline conversation, no previous_response_id)

## Agent v0.4.7
- Configurable model resolution: CLI `--model`, env `AGENT_MODEL`, config global + per-profile
- Profiles are now model-optional (resolved from config)
- `ProviderConfig.Model` and `ProviderConfig.Models` map

## Agent v0.4.6
- Token extraction: `stream_options.include_usage` for Chat Completions streaming
- Responses API: usage extraction from response body
- Switch to OpenAI Responses API (`apiMode: responses`)

## Agent v0.4.5
- Fix: model fallback stream error handling

## Agent v0.4.0–v0.4.3
- Agent memory (agent/remember, agent/recall)
- Anthropic native provider
- Reactive triggers (service-mode profiles)
- Plugin config from env (EnvVar, Default)
- Worker pod IP registration
- Model fallback chain

## Agent v0.3.0–v0.3.6
- Sub-agent orchestration (discover, spawn, parallel, pipeline)
- MCP server mode
- HTTP worker mode
- Session compaction
- On-demand plugin + profile install
- Version pinning, transitive deps

## Agent v0.1.0
- Core framework, CLI, built-in tools, OpenAI provider, sessions, policy

---

## Gateway v0.5.7
- Schedule editing via PUT /v1/schedules with dashboard edit dialog
- Cron quick-pick buttons in schedule form

## Gateway v0.5.5–v0.5.6
- Token field names fixed (inputTokens/outputTokens)
- Rich agent/plugin detail from registry (tools, capabilities, model)
- Plugins page with contributed tools display

## Gateway v0.5.3
- Token columns on tasks table (model, input_tokens, output_tokens)
- Migration for new columns

## Gateway v0.5.0–v0.5.2
- Svelte + shadcn-svelte dashboard (9 pages)
- SSE auth via query param
- /v1/plugins endpoint
- Sidebar spacing, task detail reactivity

## Gateway v0.4.0–v0.4.4
- Agent memory API, cost tracking, NATS events, web dashboard
- Dead worker eviction, retries, parallel dispatch

## Gateway v0.1.0
- Task routing, auth, webhooks, scheduling, PostgreSQL

---

## Registry v0.3.0
- Marketplace web UI (Svelte, embedded via go:embed)
- Search endpoint: GET /v1/search with filters and sort
- Detail endpoints with README content
- Download counting on artifact requests
- Stats persistence (JSON flush)
- README extraction from tarballs at publish time
- Enriched index (tools, capabilities, model, accepts/returns)

## Registry v0.2.5
- Profile spec extraction (capabilities, tools, model)
- Plugin tools and dependencies in index

## Registry v0.2.4
- Profile publish, multi-version, base-url flag
