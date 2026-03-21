# Plugin CLI Workflow

Plugins provide tools that agents use. Install from local paths, filesystem sources,
or the registry. Workers install plugins on demand when a profile needs them.

## Install

```bash
# From registry
agent plugins install ddg-research
agent plugins install ddg-research@0.2.0         # version pinning
agent plugins install ddg-research --source official

# From local path
agent plugins install ../agent-plugins/send-email --link

# Auto-installs dependencies
agent plugins install my-plugin
#   dep  grafana-alerts@0.1.0  (auto-installed)
```

## On-demand install

In k8s, workers install plugins automatically when a task requires tools
from an uninstalled plugin:

```
Task arrives → profile needs ddg/search → ddg-research not installed
  → worker searches registry → installs ddg-research → registers tools
  → task executes
```

Plugins that need config (e.g. send-email needs SMTP settings) are installed
but NOT auto-enabled. Plugins without required config are auto-enabled.

## Upgrade and publish

```bash
agent plugins upgrade ddg-research
agent plugins publish ../agent-plugins/my-plugin --registry official
```

## Config from environment

Plugins can declare `envVar` and `default` values in their config schema:

```yaml
# plugin.yaml
configSchema:
  properties:
    apiEndpoint:
      type: string
      default: "https://api.example.com"    # used if not set
    secretKey:
      type: string
      envVar: MY_SECRET_KEY                 # read from environment
      secret: true
    region:
      type: string
      default: "us-east-1"
```

At worker startup, the framework auto-populates config:
1. `Property.EnvVar` — reads the named environment variable
2. Convention: `AGENT_PLUGIN_<PLUGINNAME>_<KEY>` (uppercase)
3. `Property.Default` — fallback default value

No `plugins config set` needed in k8s — just set env vars on worker pods.

## Environment variable mapping (envMapping)

For MCP and command plugins that need env vars passed to subprocesses:

```yaml
spec:
  runtime:
    type: mcp
    command: ["mcp-grafana"]
    envMapping:
      grafanaURL: GRAFANA_URL          # config → subprocess env
      grafanaAPIKey: GRAFANA_API_KEY
```

## Dependencies

```yaml
spec:
  requires:
    plugins:
      - send-email     # must be installed and enabled
```

Dependencies are auto-installed at install time and enforced at startup.

## Plugin contributions

| Type | Auto-applied? | Override |
|---|---|---|
| `tools` | When enabled | Profile selects which tools |
| `prompts` | Opt-in by ID | Profile references or ignores |
| `profileTemplates` | Reference only | Copy and modify |
| `policies` | Opt-in | Profile lists in overlays |
| `sensitiveActions` | Yes | Profile overlay with `allow` |

## Config commands

```bash
agent plugins config set send-email provider smtp
agent plugins config set send-email smtpPort 587
agent plugins config unset send-email baseURL
agent plugins config send-email          # show config
agent plugins validate-config send-email
agent plugins enable send-email
```
