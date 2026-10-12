# bing-webmaster-ai-cli-mcp

Check whether Bing has crawled and indexed a page, and review Bing search traffic: rank and clicks over time, top queries and pages. Changes are only planned for a person to apply.

This is a community plugin for the open-source MCP server [bing-webmaster-ai-cli-mcp](https://github.com/stufently/bing-webmaster-ai-cli-mcp). It is not made or endorsed by the vendor of the service it connects to.

## Scenarios

`/bing-webmaster-ai-cli-mcp:check-page-indexing`, `/bing-webmaster-ai-cli-mcp:traffic-review`. They are skills: Claude also picks them up on its own when a request matches.

## Configuration

Claude Code asks for these values when you enable the plugin; change them later in `/config` (secret values are not shown there; re-enter them with `claude plugin configure bing-webmaster-ai-cli-mcp`). Values marked secret are stored in the system credential store, not in `settings.json`, and reach the server only through its environment.

| Setting | Key | Secret | What it is |
| --- | --- | --- | --- |
| Bing Webmaster API key | `api_key` | yes | Passed as `BING_WM_API_KEY`. |

Requires [uv](https://docs.astral.sh/uv/) (`uvx` on PATH).

## What it runs and where data goes

The plugin starts the PyPI package `bing-webmaster-ai-cli-mcp==0.1.3` with `uvx`. uv downloads the package and its dependencies from PyPI the first time. The server talks only to the Bing Webmaster API (`ssl.bing.com/webmaster/api.svc`) and, for IndexNow planning, to the IndexNow endpoint, authenticated with your API key. It sends nothing anywhere else, and the plugin has no hooks or telemetry. Plans (with their arguments) and an audit log of writes are kept on your machine in `~/.local/state/bing-webmaster-ai-cli-mcp/` until you delete them.

## Safe defaults

`BING_WM_ALLOW_WRITES=false` is set in the plugin (it is also the server's default since 0.1.3; kept explicit). Writes become `bing_plan_*` tools that only record a plan; you review and apply it yourself from a terminal with the server's CLI, `uvx --from bing-webmaster-ai-cli-mcp==0.1.3 bing-wm plan apply PLAN_ID` (`uvx` does not put `bing-wm` on your PATH; the CLI needs its own API key setup, see the [server README](https://github.com/stufently/bing-webmaster-ai-cli-mcp#readme)).

## Install

```
/plugin marketplace add stufently/claude-plugins
/plugin install bing-webmaster-ai-cli-mcp@stufently-mcp
```

## License

MIT, see [LICENSE](LICENSE).
