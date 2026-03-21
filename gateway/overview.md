# Gateway

The single entry point for the distributed agent platform. Routes tasks to workers,
handles auth, webhooks, scheduling, cost tracking, agent memory, and serves a
web dashboard.

## Endpoints

### Tasks

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/tasks` | POST | `tasks:write` | Submit task (sync or `"async":true`) |
| `/v1/tasks` | GET | `tasks:read` | List tasks (`?status=`, `?limit=`) |
| `/v1/tasks/{id}` | GET | `tasks:read` | Task details + result |
| `/v1/tasks/parallel` | POST | `tasks:write` | Multiple tasks across workers concurrently |

#### Parallel execution
```json
POST /v1/tasks/parallel
{"tasks": [
  {"profile":"researcher","task":"Anthropic news"},
  {"profile":"researcher","task":"OpenAI news"}
]}
```
Each task dispatched to a different worker. Wall time = slowest task.

### Workers

| Endpoint | Method | Description |
|---|---|---|
| `/v1/workers` | POST | Register/heartbeat (no auth — workers self-register) |
| `/v1/workers` | GET | List workers |
| `/v1/workers` | DELETE | Deregister (`?url=`) |

Workers register with pod IPs at startup (`GATEWAY_URL` env) and heartbeat every 5 minutes.
Dead workers are evicted immediately on connection failure — no 15-minute wait.

### Auth

| Endpoint | Method | Auth |
|---|---|---|
| `/v1/auth/keys` | POST/GET/DELETE | `admin` |

Scopes: `tasks:write`, `tasks:read`, `admin`. Admin key via `--admin-key` or `ADMIN_KEY`.

### Webhooks

| Endpoint | Method | Auth |
|---|---|---|
| `/v1/webhooks` | POST/GET/DELETE | `admin` |
| `/v1/webhooks/{path}` | POST | per-webhook token |

Template expansion: `{{key}}` and `{{nested.key}}` from payload.

```json
POST /v1/webhooks
{"name":"grafana","path":"grafana","profile":"alert-monitor",
 "taskTemplate":"Alert {{alertname}} on {{labels.host}}",
 "authToken":"secret"}
```

### Schedules

| Endpoint | Method | Auth |
|---|---|---|
| `/v1/schedules` | POST/GET/DELETE | `admin` |

```json
{"name":"daily-ops","cron":"0 8 * * *","timezone":"America/New_York",
 "profile":"grafana-alert-summary","task":"Daily ops report"}
```

### Memory

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/memory?profile=X` | POST | `tasks:write` | Store key-value |
| `/v1/memory?profile=X` | GET | `tasks:read` | Recall all or `&key=Y` |
| `/v1/memory?profile=X` | DELETE | `tasks:write` | Forget all or `&key=Y` |

Per-profile persistent knowledge in PostgreSQL. Agents use `agent/remember`
and `agent/recall` tools which call this API via `GATEWAY_URL`.

### Costs

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/costs` | GET | `tasks:read` | Cost summary by profile (`?since=2026-01-01`) |
| `/v1/costs/pricing` | GET | `admin` | View model pricing table |
| `/v1/costs/pricing` | POST | `admin` | Set per-model pricing |

Pricing synced from [models.dev](https://models.dev) at startup (1800+ models,
USD per million tokens). Admin can override any model's pricing.

Token usage flows: LLM response → provider → runner → worker HTTP response →
gateway → `cost_tracking` table → `GET /v1/costs`.

### Events and dashboard

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/events` | GET | `tasks:read` | SSE event stream |
| `/v1/agents` | GET | `tasks:read` | Available agents (workers + registry) |
| `/v1/health` | GET | none | Health check |
| `/` | GET | none | Web dashboard |

NATS events published: `agent.task.submitted`, `agent.task.started`,
`agent.task.completed`, `agent.task.failed`, `agent.task.retry`,
`agent.worker.joined`, `agent.webhook.fired`.

## Routing

1. Find idle worker with profile installed
2. Fall back to any idle worker (on-demand install handles it)
3. If worker unreachable → evict immediately, retry on different worker
4. Up to 2 retries on transient errors (timeouts, 502/503)
5. Permanent errors (auth, invalid model) fail immediately

## Reactive triggers

Service-mode profiles register their triggers as webhooks at worker startup:

```yaml
# profile with mode: service
spec:
  mode: service
  triggers:
    - event: agent.alert.fired
      taskTemplate: "Alert {{alertname}}. Investigate."
```

Worker starts → discovers service-mode profiles → registers webhook on gateway →
external system fires event → gateway creates task → worker executes agent.

## Configuration

```
--addr          :8080
--dsn           postgres://... (or DATABASE_URL)
--nats          nats://... (or NATS_URL, optional)
--registry      http://registry:9080
--admin-key     ... (or ADMIN_KEY)
```

## PostgreSQL tables

`tasks`, `workers`, `api_keys`, `schedules`, `webhooks`, `agent_memory`,
`cost_tracking`, `audit_log`

Auto-migrated on startup.

## Docker

```bash
docker run -p 8080:8080 ghcr.io/bitop-dev/agent-gateway:0.4.4 \
  --dsn "postgres://..." --nats "nats://nats:4222" --admin-key "..."
```
