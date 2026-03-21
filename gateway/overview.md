# Gateway

The agent gateway is the single entry point for the distributed agent platform.
It accepts tasks from external clients, routes them to workers, and provides
auth, webhooks, scheduled tasks, and task history.

## Architecture

```
External clients → Gateway → Workers
                      ↕
                   PostgreSQL
                      ↕
                   Registry (profile + plugin discovery)
```

The gateway owns:
- **Task lifecycle** — queued → running → completed/failed
- **Worker management** — registration, health, load-aware routing
- **Authentication** — API keys with scopes
- **Webhooks** — external triggers (Grafana, Slack, GitHub) → tasks
- **Scheduling** — cron-based recurring tasks
- **Task history** — all results stored in PostgreSQL

## Endpoints

### Tasks
- `POST /v1/tasks` — submit a task (sync or async)
- `GET /v1/tasks` — list tasks with optional `?status=` filter
- `GET /v1/tasks/{id}` — get task details and result

### Workers
- `POST /v1/workers` — register/heartbeat a worker
- `GET /v1/workers` — list workers
- `DELETE /v1/workers?url=` — deregister

### Auth
- `POST /v1/auth/keys` — create API key (admin)
- `GET /v1/auth/keys` — list keys (admin)
- `DELETE /v1/auth/keys?id=` — revoke key (admin)

### Webhooks
- `POST /v1/webhooks` — create webhook config (admin)
- `GET /v1/webhooks` — list webhooks (admin)
- `POST /v1/webhooks/{path}` — trigger (per-webhook auth)

### Schedules
- `POST /v1/schedules` — create schedule (admin)
- `GET /v1/schedules` — list schedules (admin)
- `DELETE /v1/schedules?id=` — delete schedule (admin)

### Discovery
- `GET /v1/agents` — available agents (from workers + registry)
- `GET /v1/health` — health check

## Auth model

Three levels of auth:
1. **Admin key** — full access, set via `--admin-key` or `ADMIN_KEY` env
2. **API keys** — created by admin, scoped to `tasks:write`, `tasks:read`, or `admin`
3. **Webhook tokens** — per-webhook auth tokens for external triggers

Unauthenticated endpoints: `/v1/health`, `/v1/workers` (workers self-register)

## Webhooks

Configure a webhook to map external events to agent tasks:

```json
POST /v1/webhooks
{
  "name": "grafana-alerts",
  "path": "grafana",
  "profile": "grafana-researcher",
  "taskTemplate": "Alert: {{alertname}} on {{labels.host}}. Investigate.",
  "contextTemplate": {"team": "{{labels.team}}"},
  "authToken": "grafana-webhook-secret"
}
```

Then configure Grafana/Slack/GitHub to POST to:
```
POST https://gateway.example.com/v1/webhooks/grafana
Authorization: Bearer grafana-webhook-secret
```

The gateway expands `{{key}}` and `{{nested.key}}` from the incoming payload,
creates a task, and dispatches it to a worker asynchronously.

## Scheduled tasks

Create recurring agent tasks:

```json
POST /v1/schedules
{
  "name": "daily-ops-report",
  "cron": "0 8 * * *",
  "timezone": "America/New_York",
  "profile": "grafana-alert-summary",
  "task": "Generate daily ops report for ict-aipe. Send to nick@bitop.dev"
}
```

The scheduler checks every 30 seconds for due schedules and creates tasks automatically.

## Task routing

When a task arrives, the gateway:
1. Looks for idle workers with the requested profile installed
2. Falls back to any idle worker (on-demand profile install handles the rest)
3. Marks the worker as busy during execution
4. Stores the result in PostgreSQL
5. Clears the worker for the next task

Workers that haven't sent a heartbeat in 15 minutes are marked stale.

## Deployment

```yaml
# Docker
docker run -p 8080:8080 ghcr.io/bitop-dev/agent-gateway:0.1.0 \
  --dsn "postgres://..." --registry "http://registry:9080" --admin-key "..."

# k8s — see agent-deploy/k8s/gateway.yaml
```

Requires PostgreSQL for state storage. The gateway runs migrations
automatically on startup.
