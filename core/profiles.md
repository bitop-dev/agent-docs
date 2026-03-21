# Profiles

A **profile** is the complete definition of an agent — its model, tools, system prompt,
behavior, and identity. Profiles are YAML files published to the registry and pulled
by workers on demand.

## Full profile reference

```yaml
apiVersion: agent/v1
kind: Profile
metadata:
  name: my-agent
  version: 0.1.0
  description: What this agent does
  extends: base-researcher            # inherit from another profile
  capabilities: [web-search, summarization]  # discovery tags
  accepts: A topic to research        # what input this agent expects
  returns: Structured summary         # what output it produces

spec:
  mode: oneshot                       # oneshot (default) or service
  triggers:                           # for service mode — events that activate this agent
    - event: agent.alert.fired
      taskTemplate: "Alert {{alertname}} on {{host}}. Investigate."

  instructions:
    system:
      - ./prompts/system.md           # file path
      - email/style-default           # plugin prompt ID
      - "Always cite sources."        # inline text

  provider:
    default: openai                   # openai or anthropic
    model: gpt-4o
    fallback:                         # tried in order if primary fails
      - gpt-4o-mini
      - gpt-3.5-turbo

  tools:
    enabled:
      - ddg/search
      - ddg/fetch
      - agent/remember
      - agent/recall

  approval:
    mode: on-request                  # never | always | on-request
    requireFor: [email/send]

  workspace:
    required: false
    writeScope: read-only             # workspace | read-only

  session:
    persistence: sqlite               # sqlite | none
    compaction: auto                   # auto | none

  policy:
    overlays:
      - ./policies/strict.yaml
```

## Discovery metadata

Profiles declare capabilities so orchestrators can find them dynamically:

```yaml
metadata:
  capabilities: [web-search, grafana-alerts, email]
  accepts: A team name and time range
  returns: Structured alert summary
```

The `agent/discover` tool returns all available profiles — both locally installed
and from the registry — with their capabilities, accepts, and returns fields.

## Profile inheritance

Profiles can extend other profiles:

```yaml
metadata:
  name: security-researcher
  extends: researcher       # inherits tools, instructions, provider from researcher
spec:
  instructions:
    system:
      - "Focus on cybersecurity news."  # appended after parent instructions
  tools:
    enabled:
      - cve/search          # added to parent's tools (union)
```

Merge rules:
- **Tools** — union of parent and child
- **Instructions** — parent first, child appended
- **Provider/approval/workspace/session** — child overrides parent
- **Capabilities** — child overrides if set

## Model fallback chain

If the primary model fails, the runner tries each fallback in order:

```yaml
provider:
  model: gpt-4o
  fallback:
    - gpt-4o-mini          # tried if gpt-4o fails
    - gpt-3.5-turbo        # tried if gpt-4o-mini also fails
```

Permanent model errors (400 Bad Request, invalid model name) skip immediately
to the next fallback without retrying. Transient errors (timeouts) retry the
same model first.

## Providers

Two providers are supported:

```yaml
# OpenAI-compatible (default) — works with any proxy
provider:
  default: openai
  model: gpt-4o

# Native Anthropic Messages API
provider:
  default: anthropic
  model: claude-sonnet-4-5
```

Configure in `~/.agent/config.yaml`:
```yaml
providers:
  openai:
    baseURL: https://api.openai.com/v1
    apiKey: sk-...
  anthropic:
    apiKey: sk-ant-...
```

Or via environment: `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`.

## Agent memory

Agents can persist facts across tasks using `agent/remember` and `agent/recall`:

```yaml
tools:
  enabled:
    - agent/remember     # store key-value facts
    - agent/recall       # retrieve stored facts
```

Memory is scoped per-profile and stored in PostgreSQL via the gateway.
Requires `GATEWAY_URL` environment variable on the worker.

Example agent behavior:
```
[tool request] agent/recall
→ last_report_date = 2026-03-20

[tool request] agent/remember
→ remembered: last_report_date = 2026-03-21
```

## Reactive triggers (service mode)

Profiles with `mode: service` declare event triggers that automatically create tasks:

```yaml
spec:
  mode: service
  triggers:
    - event: agent.alert.fired
      taskTemplate: "Alert {{alertname}} on {{host}}. Investigate."
    - event: agent.task.failed
      taskTemplate: "Task {{taskId}} failed. Diagnose the issue."
```

When a worker starts with this profile, it registers the triggers as webhooks
on the gateway. External systems fire events via `POST /v1/webhooks/{path}`,
and the gateway creates tasks automatically.

## MCP server mode

Any profile can be exposed as an MCP tool for external clients:

```bash
agent serve --profile researcher
```

Configure in opencode/Claude Desktop:
```json
{
  "mcp": {
    "researcher": {
      "type": "local",
      "command": ["agent", "serve", "--profile", "researcher"],
      "timeout": 120000
    }
  }
}
```

## On-demand loading

Workers pull profiles from the registry when a task arrives:

```
Task: {profile: "researcher", task: "..."}
  → Worker checks ~/.agent/profiles/researcher/ — not found
  → Worker queries registry /v1/profiles/researcher.json
  → Downloads and extracts profile tarball
  → Loads profile, resolves tools, installs plugins
  → Executes task
  → Profile cached for next request
```

No pre-installation needed. Publish to registry, workers pull on demand.

## Profile locations

| Location | Purpose |
|---|---|
| `~/.agent/profiles/<name>/` | User-level, permanent |
| `.agent/profiles/<name>/` | Project-local |
| Registry | Published profiles, pulled on demand |

## Publishing

```bash
agent profiles publish ./my-profile --registry official
```

Or directly via the registry API:
```bash
tar czf - my-profile | curl -X POST http://registry:9080/v1/profiles \
  -H "Authorization: Bearer <token>" --data-binary @-
```
