# Marketplace

The registry serves a browsable marketplace UI at `/` for discovering plugins
and profiles. It also provides search and detail APIs.

## Web UI

Visit the registry URL in a browser to browse:

```
http://registry:9080/              — Landing page with search + popular
http://registry:9080/#/plugins     — Browse all plugins
http://registry:9080/#/profiles    — Browse all profiles
http://registry:9080/#/plugins/github  — Plugin detail with README + install command
http://registry:9080/#/profiles/researcher — Profile detail
http://registry:9080/#/search?q=kubernetes — Search results
```

Dark/light mode toggle. All package cards show download counts.

## API endpoints

### Search
```
GET /v1/search?q=github&type=plugin&category=integration&sort=downloads
```

Parameters:
- `q` — search query (matches name, description, keywords, tools, capabilities)
- `type` — `plugin`, `profile`, or empty for both
- `category` — filter by category
- `runtime` — filter by runtime type
- `sort` — `downloads` or `name` (default)

Response:
```json
{
  "results": [
    {
      "name": "github",
      "type": "plugin",
      "version": "0.1.0",
      "description": "GitHub integration...",
      "tools": ["github/repo", "github/issues", ...],
      "downloads": 42
    }
  ],
  "count": 1,
  "query": "github"
}
```

### Plugin detail
```
GET /v1/packages/{name}/detail.json
```

Returns full metadata including README:
```json
{
  "name": "github",
  "type": "plugin",
  "version": "0.1.0",
  "description": "...",
  "tools": ["github/repo", ...],
  "dependencies": [],
  "downloads": 42,
  "readme": "# github\n\nGitHub integration via..."
}
```

### Profile detail
```
GET /v1/profiles/{name}/detail.json
```

Returns full profile spec including README:
```json
{
  "name": "researcher",
  "type": "profile",
  "version": "0.1.0",
  "description": "...",
  "capabilities": ["web-search", "summarization"],
  "tools": ["ddg/search", "ddg/fetch"],
  "accepts": "A topic to research",
  "returns": "Structured summary",
  "downloads": 15,
  "readme": "..."
}
```

### Download counts

Downloads are tracked automatically when artifacts are fetched. Counts
are included in index responses and search results. Stats persist to
`data/stats.json` and flush every 60 seconds.

## Multiple registries

Agents support multiple registry sources. Configure in `~/.agent/config.yaml`:

```yaml
pluginSources:
  - name: community
    type: registry
    url: https://registry.bitop.dev    # public marketplace
    
  - name: internal
    type: registry
    url: http://registry.internal:9080  # private org registry
    publishToken: corp-token-abc
```

Workers search all configured sources. Install from a specific source:
```bash
agent plugins install github --source community
```
