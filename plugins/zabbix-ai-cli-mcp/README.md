# zabbix-ai-cli-mcp

Ask Zabbix what is broken right now, investigate one alert end to end, and find out why a notification did or did not arrive. Read-only: the bundled server starts with writes switched off, so it offers no tool that changes Zabbix.

This is a community plugin for the open-source MCP server [zabbix-ai-cli-mcp](https://github.com/stufently/zabbix-ai-cli-mcp). It is not made or endorsed by the vendor of the service it connects to.

## Scenarios

`/zabbix-ai-cli-mcp:investigate-alert`, `/zabbix-ai-cli-mcp:whats-broken`. They are skills: Claude also picks them up on its own when a request matches.

## Configuration

Claude Code asks for these values when you enable the plugin; change them later in `/config` (secret values are not shown there; re-enter them with `claude plugin configure zabbix-ai-cli-mcp`). Values marked secret are stored in the system credential store, not in `settings.json`, and reach the server only through its environment.

| Setting | Key | Secret | What it is |
| --- | --- | --- | --- |
| Zabbix URL | `zabbix_url` | no | Base URL of your Zabbix frontend; passed as `ZABBIX_AI_CLI_MCP_URL`. |
| Zabbix API token | `zabbix_token` | yes | Passed as `ZABBIX_AI_CLI_MCP_TOKEN`. Prefer a token of a user with read-only rights. |

Requires Docker.

## What it runs and where data goes

The plugin starts the container image `ghcr.io/stufently/zabbix-ai-cli-mcp:0.4.3`, pinned by digest, with `docker run -i --rm`. Docker pulls the image from GitHub Container Registry (`ghcr.io`) the first time. The server talks only to the Zabbix URL you enter, over the Zabbix JSON-RPC API, authenticated with your API token. It sends nothing anywhere else, and the plugin has no hooks or telemetry.

## Safe defaults

`ZABBIX_AI_CLI_MCP_ALLOW_WRITE=false` is fixed in the plugin. The server would allow direct writes otherwise; with it off, `zabbix_write` is not offered. To let an agent change Zabbix, configure the server yourself instead of using this plugin.

## Install

```
/plugin marketplace add stufently/claude-plugins
/plugin install zabbix-ai-cli-mcp@stufently-mcp
```

## License

MIT, see [LICENSE](LICENSE).
