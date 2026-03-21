# Gateway

The agent gateway is the single entry point for the distributed agent platform.
It accepts tasks, routes them to workers, retries on failures, stores results
in PostgreSQL, and provides auth, webhooks, scheduling, and a web dashboard.

## Architecture

```
External clients → Gateway → Workers (distributed across k8s pods)
                      ↕           ↕
                 PostgreSQL    Registry (profiles + plugins)
                      ↕
                    NATS (real-time events)
```

## Endpoints

### Tasks

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/tasks` | POST | `tasks:write` | Submit a task (sync or async) |
| `/v1/tasks` | GET | `tasks:read` | List tasks (`?status=`, `?limit=`) |
| `/v1/tasks/{id}` | GET | `tasks:read` | Get task details + result |
| `/v1/tasks/parallel` | POST | `tasks:write` | Submit multiple tasks for parallel execution |

#### Single task

```json
POST /v1/tasks
{"profile": "researcher", "task": "Research Anthropic news"}
→ {"id": "task-abc", "status": "completed", "output": "...", "durationMs": 12000}
```

#### Async task

```json
POST /v1/tasks
{"profile": "researcher", "task": "...", "async": true}
→ {"id": "task-abc", "status": "queued"}

GET /v1/tasks/task-abc  (poll until completed)
→ {"id": "task-abc", "status": "completed", "output": "..."}
```

#### Parallel tasks (distributed across workers)

Submits multiple tasks and dispatches each to a different worker concurrently.
Total wall time equals the slowest task, not the sum.

```json
POST /v1/tasks/parallel
{
  "tasks": [
    {"profile": "researcher", "task": "Anthropic news"},
    {"profile": "researcher", "task": "OpenAI news"},
    {"profile": "researcher", "task": "Google AI news"}
  ]
}
→ {"count": 3, "tasks": [{...}, {...}, {...}]}
```

Each task is routed to a different worker. With 5 workers and 3 tasks,
3 different pods handle the work concurrently.

### Retries

The gateway automatically retries failed tasks on transient errors:
- Timeouts, connection refused, EOF, 502/503/504
- Up to 2 retries (configurable)
- Each retry picks a different worker if available
- Exponential backoff (2s × attempt) between retries
- Permanent errors (auth failures, missing tools) are not retried

### Workers

| Endpoint | Method | Description |
|---|---|---|
| `/v1/workers` | POST | Register/heartbeat |
| `/v1/workers` | GET | List workers |
| `/v1/workers` | DELETE | Deregister (`?url=`) |

Workers heartbeat every 5 minutes. Stale workers (no heartbeat for 15 min)
are marked inactive and excluded from routing.

### Auth

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/auth/keys` | POST | `admin` | Create API key |
| `/v1/auth/keys` | GET | `admin` | List keys |
| `/v1/auth/keys` | DELETE | `admin` | Revoke (`?id=`) |

Three auth levels:
1. **Admin key** — `--admin-key` flag or `ADMIN_KEY` env. Full access.
2. **API keys** — scoped to `tasks:write`, `tasks:read`, or `admin`
3. **Webhook tokens** — per-webhook auth for external triggers

### Webhooks

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/webhooks` | POST | `admin` | Create webhook config |
| `/v1/webhooks` | GET | `admin` | List webhooks |
| `/v1/webhooks/{path}` | POST | webhook token | Trigger |

Template expansion with `{{key}}` and `{{nested.key}}`:

```json
POST /v1/webhooks
{
  "name": "grafana-alerts",
  "path": "grafana",
  "profile": "grafana-researcher",
  "taskTemplate": "Alert: {{alertname}} on {{labels.host}}. Investigate.",
  "contextTemplate": {"team": "{{labels.team}}"},
  "authToken": "grafana-secret"
}

// Trigger from Grafana:
POST /v1/webhooks/grafana
Authorization: Bearer grafana-secret
{"alertname": "High CPU", "labels": {"host": "prod-01", "team": "ops"}}
→ {"taskId": "task-xyz", "status": "queued"}
```

### Schedules

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/schedules` | POST | `admin` | Create schedule |
| `/v1/schedules` | GET | `admin` | List schedules |
| `/v1/schedules` | DELETE | `admin` | Delete (`?id=`) |

```json
POST /v1/schedules
{
  "name": "daily-ops",
  "cron": "0 8 * * *",
  "timezone": "America/New_York",
  "profile": "grafana-alert-summary",
  "task": "Daily ops report for ict-aipe"
}
```

### Discovery and events

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/agents` | GET | `tasks:read` | Available agents (workers + registry) |
| `/v1/events` | GET | `tasks:read` | SSE event stream (real-time) |
| `/v1/health` | GET | none | Health check |

### Web dashboard

The gateway serves an embedded web dashboard at `/`. Connect with an API key
to see:
- Worker status and count
- Task history with status, duration, profile
- Agent discovery
- Live event feed

## Task routing

When a task arrives:
1. Find idle workers with the profile installed
2. Fall back to any idle worker (on-demand install handles the rest)
3. Prefer workers with fewer completed tasks (load balance)
4. If no worker available, retry with backoff
5. Store result in PostgreSQL on completion

## Distributed parallel execution

Workers set `GATEWAY_URL=http://agent-gateway:8080` to enable gateway-distributed
parallelism. When `agent/spawn-parallel` is called by an orchestrator:
- Without GATEWAY_URL: goroutines in one worker (serializes on LLM API)
- With GATEWAY_URL: dispatched through `/v1/tasks/parallel` across pods

This means an orchestrator running on Worker A can dispatch sub-tasks to
Workers B, C, D concurrently for true parallel execution.

## Configuration

```
--addr          Listen address (default :8080)
--dsn           PostgreSQL connection string (or DATABASE_URL env)
--nats          NATS URL (or NATS_URL env, optional)
--registry      agent-registry URL for profile discovery
--admin-key     Admin API key (or ADMIN_KEY env)
```

## NATS events

When NATS is configured, the gateway publishes events:
- `agent.task.submitted` / `started` / `completed` / `failed` / `retry`
- `agent.worker.joined` / `lost`
- `agent.webhook.fired`
- `agent.schedule.fired`

Subscribe to `agent.>` for all events.

## Deployment

```bash
# Docker
docker run -p 8080:8080 ghcr.io/bitop-dev/agent-gateway:0.3.2 \
  --dsn "postgres://..." --nats "nats://nats:4222" --admin-key "..."

# k8s (6 pods total)
# gateway (1) + workers (N) + registry (1) + postgres (1) + nats (1)
```
