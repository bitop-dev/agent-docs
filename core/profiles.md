# Profiles

A **profile** defines an agent — its tools, system prompt, behavior, and identity.
Profiles are YAML files published to the registry and pulled by workers on demand.
Profiles are **model-optional** — the model is resolved from config, not hardcoded.

## Full profile reference

```yaml
apiVersion: agent/v1
kind: Profile
metadata:
  name: my-agent
  version: 0.1.0
  description: What this agent does
  extends: base-researcher            # inherit from another profile
  capabilities: [web-search, summarization]
  accepts: A topic to research
  returns: Structured summary

spec:
  instructions:
    system:
      - ./prompts/system.md           # file path
      - "Always cite sources."        # inline text

  provider:
    default: openai                   # openai or anthropic
    # model is OPTIONAL — resolved from config (see Model Resolution below)
    # model: gpt-4o                   # only set if this profile needs a specific model
    fallback:                         # tried in order if primary fails
      - gpt-4o-mini

  tools:
    enabled:
      - ddg/search
      - ddg/fetch
      - agent/remember
      - agent/recall

  approval:
    mode: never                       # never | always | on-request

  workspace:
    required: false
    writeScope: read-only

  session:
    persistence: none                 # sqlite | none
    compaction: auto

  policy:
    overlays:
      - ./policies/allow-spawn.yaml
```

## Model resolution

Profiles don't need to specify a model. The model is resolved with this priority
(highest wins):

```
1. CLI flag:              agent run --model gpt-4o-mini
2. Env var:               AGENT_MODEL=gpt-4o-mini
3. Config per-profile:    providers.openai.models.code-reviewer: claude-sonnet-4-5
4. Config global:         providers.openai.model: gpt-4o
5. Profile YAML:          spec.provider.model: gpt-4o (publisher recommendation)
6. Hardcoded fallback:    gpt-4o
```

Configuration example:
```yaml
# ~/.agent/config.yaml
providers:
  openai:
    baseURL: https://api.openai.com/v1
    apiKey: sk-...
    model: gpt-4o                    # default for all profiles
    apiMode: responses
    models:                          # per-profile overrides
      code-reviewer: claude-sonnet-4-5
      writer: gpt-4o-mini
```

This means:
- Published profiles work with any provider — no model lock-in
- Users set their preferred model once in config
- Different profiles can use different models via `models:` map
- CLI override for one-off runs: `--model gpt-4o-mini`

## Profile inheritance

```yaml
metadata:
  name: security-researcher
  extends: researcher
spec:
  instructions:
    system:
      - "Focus on cybersecurity news."  # appended after parent
```

Merge rules:
- **Tools** — union of parent and child
- **Instructions** — parent first, child appended
- **Provider/approval/workspace/session** — child overrides parent

## Available profiles

| Profile | Tools | Purpose |
|---|---|---|
| **researcher** | ddg/search, ddg/fetch | Web research with structured summaries |
| **orchestrator** | agent/discover, agent/spawn, agent/spawn-parallel, agent/pipeline, agent/remember, agent/recall | Multi-agent coordinator |
| **security-researcher** | (inherits researcher) | CVE/vulnerability research |
| **code-reviewer** | github/repo, github/pulls, github/pr-diff, github/file, ddg/search | PR code review |
| **devops** | k8s/*, docker/*, ddg/search | K8s + Docker troubleshooting |
| **writer** | ddg/search, ddg/fetch | Research and polished content |

## Publishing

```bash
tar czf - my-profile | curl -X POST http://registry:9080/v1/profiles \
  -H "Authorization: Bearer <token>" --data-binary @-
```
