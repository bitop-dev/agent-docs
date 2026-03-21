# Plugin Catalog

9 plugins available in the marketplace. Workers install plugins on demand
when a profile needs them — no pre-installation required.

## Integration plugins

### ddg-research
**Web search via DuckDuckGo** — no API key required.

| Tool | Description |
|------|-------------|
| `ddg/search` | Search the web. Returns titles, URLs, snippets. |
| `ddg/fetch` | Fetch and extract readable content from a URL. |

Runtime: Go binary (`command`). [README](https://github.com/bitop-dev/agent-plugins/tree/main/ddg-research)

---

### github
**GitHub API integration** — repos, issues, PRs, diffs, search, file contents.

| Tool | Description |
|------|-------------|
| `github/repo` | Repository info (stars, forks, language) |
| `github/issues` | List issues with state, labels |
| `github/pulls` | List pull requests |
| `github/pr-diff` | Full diff for a pull request |
| `github/search` | Search repos, code, issues, users |
| `github/file` | Get file contents from a repo |

Runtime: Go binary (`command`). Config: `GITHUB_TOKEN` for private repos.

---

### http-request
**Generic HTTP client** — call any REST API.

| Tool | Description |
|------|-------------|
| `http/request` | GET, POST, PUT, PATCH, DELETE with headers, auth, JSON parsing |

Runtime: Go binary (`command`). Supports bearer tokens, custom headers, TLS skip.

---

### csv-tool
**CSV/TSV data processing** — parse, filter, query.

| Tool | Description |
|------|-------------|
| `csv/parse` | Parse CSV into JSON, markdown table, or summary |
| `csv/query` | Filter rows, select columns, sort |

Runtime: Go binary (`command`).

---

### send-email
**Email via SMTP** — draft and send.

| Tool | Description |
|------|-------------|
| `email/draft` | Draft an email (preview without sending) |
| `email/send` | Send via configured SMTP server |

Runtime: HTTP. Config: SMTP host, port, credentials.

---

### slack
**Slack messaging** — post to channels.

| Tool | Description |
|------|-------------|
| `slack/post-message` | Plain text message |
| `slack/post-blocks` | Rich Block Kit message |

Runtime: Go binary (`command`). Config: webhook URL or API token.

## Infrastructure plugins

### kubectl
**Kubernetes CLI** — cluster operations.

| Tool | Description |
|------|-------------|
| `k8s/get-pods` | List pods |
| `k8s/get-deployments` | List deployments |
| `k8s/get-events` | List events |
| `k8s/logs` | View pod logs |
| `k8s/get-namespaces` | List namespaces |
| `k8s/describe` | Describe any resource |

Runtime: command (wraps `kubectl`). Requires kubectl configured on host.

---

### docker
**Docker container operations** — read-only.

| Tool | Description |
|------|-------------|
| `docker/ps` | List containers |
| `docker/logs` | View container logs |
| `docker/inspect` | Container/image details |
| `docker/images` | List images |

Runtime: Go binary (`command`). Requires docker CLI on host.

## Orchestration plugins

### spawn-sub-agent
**Multi-agent orchestration** — core framework plugin.

| Tool | Description |
|------|-------------|
| `agent/discover` | Find available agent profiles and capabilities |
| `agent/spawn` | Spawn a sub-agent with a specific profile |
| `agent/spawn-parallel` | Run multiple sub-agents concurrently |
| `agent/pipeline` | Chain agents in sequence with variable routing |
| `agent/remember` | Store a fact in persistent agent memory |
| `agent/recall` | Retrieve facts from agent memory |

Runtime: host (built into agent framework).
