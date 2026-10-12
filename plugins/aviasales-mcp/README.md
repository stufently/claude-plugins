# aviasales-mcp

Find the cheapest flight for a route, a month or a budget from Aviasales (Travelpayouts) cached fares, or ask where you can fly cheaply from a city. Read-only: nothing books or pays.

This is a community plugin for the open-source MCP server [aviasales-mcp](https://github.com/stufently/aviasales-mcp). It is not made or endorsed by the vendor of the service it connects to.

## Scenarios

`/aviasales-mcp:find-cheap-flight`, `/aviasales-mcp:where-to-fly`. They are skills: Claude also picks them up on its own when a request matches.

## Configuration

Claude Code asks for these values when you enable the plugin; change them later in `/config` (secret values are not shown there; re-enter them with `claude plugin configure aviasales-mcp`). Values marked secret are stored in the system credential store, not in `settings.json`, and reach the server only through its environment.

| Setting | Key | Secret | What it is |
| --- | --- | --- | --- |
| Travelpayouts API token | `api_token` | yes | Passed as `AVIASALES_API_TOKEN`. |
| Market | `market` | no | Two-letter market of the fare cache (default `ru`); passed as `AVIASALES_MARKET`. |

Requires Docker.

## What it runs and where data goes

The plugin starts the container image `ghcr.io/stufently/aviasales-mcp:0.5.2`, pinned by digest, with `docker run -i --rm`. Docker pulls the image from GitHub Container Registry (`ghcr.io`) the first time. The container sees only the two variables above: not your shell environment and not a `.env` file in your project. The server talks only to the Travelpayouts data API (`api.travelpayouts.com`), authenticated with your API token. Booking links it returns point to aviasales.ru. It sends nothing anywhere else, and the plugin has no hooks or telemetry.

## Safe defaults

Every tool reads cached fares or reference data. The API has no booking or payment call.

## Install

```
/plugin marketplace add stufently/claude-plugins
/plugin install aviasales-mcp@stufently-mcp
```

## License

MIT, see [LICENSE](LICENSE).
