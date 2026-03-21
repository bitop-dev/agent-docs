# Gateway

Single entry point for the distributed agent platform. Routes tasks to workers,
handles auth, webhooks, scheduling, cost tracking, agent memory, and serves
a web dashboard.

## Endpoints

| Endpoint | Method | Auth | Description |
|---|---|---|---|
| `/v1/tasks` | POST | `tasks:write` | Submit task (sync or async) |
| `/v1/tasks` | GET | `tasks:read` | List tasks |
| `/v1/tasks/{id}` | GET | `tasks:read` | Task details + result + tokens |
| `/v1/tasks/parallel` | POST | `tasks:write` | Parallel across workers |
| `/v1/workers` | POST/GET/DELETE | none | Worker registration |
| `/v1/auth/keys` | POST/GET/DELETE | `admin` | API key management |
| `/v1/webhooks` | POST/GET/PUT/DELETE | `admin` | Webhook CRUD |
| `/v1/webhooks/{path}` | POST | webhook token | Trigger webhook |
| `/v1/schedules` | POST/GET/PUT/DELETE | `admin` | Schedule CRUD with edit |
| `/v1/memory?profile=X` | POST/GET/DELETE | `tasks:*` | Agent memory |
| `/v1/costs` | GET | `tasks:read` | Cost summary by profile |
| `/v1/costs/pricing` | GET/POST | `admin` | Model pricing (models.dev) |
| `/v1/agents` | GET | `tasks:read` | Available agents with full spec |
| `/v1/plugins` | GET | `tasks:read` | Available plugins with tools |
| `/v1/events` | GET | `tasks:read` | SSE event stream |
| `/v1/health` | GET | none | Health check |
| `/` | GET | none | Web dashboard |

## Dashboard

Svelte + shadcn-svelte web dashboard at `/`:

- **Overview** — stat cards, recent tasks, live SSE events
- **Tasks** — list with filters, detail with output/tokens/model, submit form
- **Workers** — card grid with status
- **Agents** — profile cards with capabilities, tools, model. Click for detail dialog.
- **Plugins** — plugin cards with contributed tools. Click for detail dialog.
- **Costs** — time range filters, by-profile breakdown, model pricing table
- **Memory** — per-profile key-value browser with CRUD
- **Webhooks** — list, create, copy trigger URL, delete
- **Schedules** — list, create, **edit**, toggle enabled, cron quick-picks

## Task data

Tasks now include model and token data:
```json
{
  "id": "task-abc123",
  "profile": "researcher",
  "status": "completed",
  "model": "gpt-4o",
  "inputTokens": 774,
  "outputTokens": 49,
  "durationMs": 5100,
  "output": "...",
  "workerUrl": "http://10.244.1.95:9898"
}
```

## SSE auth

SSE EventSource can't set headers. Pass token as query param:
```
GET /v1/events?token=<api-key>
```

## Configuration

```
--addr          :8080
--dsn           postgres://... (or DATABASE_URL)
--nats          nats://... (or NATS_URL)
--registry      http://registry:9080
--admin-key     ... (or ADMIN_KEY)
```
