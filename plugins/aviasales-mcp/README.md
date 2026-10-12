# aviasales-mcp

Claude Code plugin for [aviasales-mcp](https://github.com/stufently/aviasales-mcp). See the [marketplace README](../../README.md) for install, environment variables and safe defaults.

## What it runs and where it connects

The plugin starts the container image `ghcr.io/stufently/aviasales-mcp:0.5.1` with `docker run`. The server talks only to the Travelpayouts (Aviasales) data API with your `AVIASALES_API_TOKEN`. Credentials come from environment variables in the shell that starts Claude Code; the plugin stores none. Every tool reads cached fares or reference data; nothing books or pays.

Scenarios: `/aviasales-mcp:find-cheap-flight`, `/aviasales-mcp:where-to-fly`

Install:

```
/plugin install aviasales-mcp --marketplace stufently/claude-plugins
```

or from a shell: `claude plugin marketplace add stufently/claude-plugins && claude plugin install aviasales-mcp@stufently-mcp`.
