# yandex-metrica-mcp

Traffic and goal-conversion reports from Yandex Metrica for one counter: visits, users, bounce rate and depth by day and source, and reaches and conversion per goal.

This is a community plugin for the open-source MCP server [yandex-metrica-mcp](https://github.com/stufently/yandex-metrica-mcp). It is not made or endorsed by the vendor of the service it connects to.

## Scenarios

`/yandex-metrica-mcp:traffic-report`, `/yandex-metrica-mcp:goal-conversions`. They are skills: Claude also picks them up on its own when a request matches.

## Configuration

Claude Code asks for these values when you enable the plugin; change them later in `/config` (secret values are not shown there; re-enter them with `claude plugin configure yandex-metrica-mcp`). Values marked secret are stored in the system credential store, not in `settings.json`, and reach the server only through its environment.

| Setting | Key | Secret | What it is |
| --- | --- | --- | --- |
| Yandex Metrica OAuth token | `oauth_token` | yes | Passed as `YANDEX_METRICA_TOKEN`. |

Requires Docker.

## What it runs and where data goes

The plugin starts the container image `ghcr.io/stufently/yandex-metrica-mcp:0.2.2`, pinned by digest, with `docker run -i --rm`. Docker pulls the image from GitHub Container Registry (`ghcr.io`) the first time. The server talks only to the Yandex Metrica Management, Reporting and Logs APIs (`api-metrika.yandex.net`), authenticated with your OAuth token. It sends nothing anywhere else, and the plugin has no hooks or telemetry.

## Safe defaults

The server has no read-only switch and also exposes counter create/delete tools. Issue the OAuth token with the `metrika:read` scope only; those calls then fail at the API. The scenarios never call them.

## Install

```
/plugin marketplace add stufently/claude-plugins
/plugin install yandex-metrica-mcp@stufently-mcp
```

## License

MIT, see [LICENSE](LICENSE).
