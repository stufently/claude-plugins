# yandex-metrica-mcp

Claude Code plugin for [yandex-metrica-mcp](https://github.com/stufently/yandex-metrica-mcp). See the [marketplace README](../../README.md) for install, environment variables and safe defaults.

## What it runs and where it connects

The plugin starts the container image `ghcr.io/stufently/yandex-metrica-mcp:0.2.2` with `docker run`. The server talks only to the Yandex Metrica Management, Reporting and Logs APIs with your `YANDEX_METRICA_TOKEN`. Credentials come from environment variables in the shell that starts Claude Code; the plugin stores none. The server also has counter create/delete tools; the scenarios never call them. Issue the token with the `metrika:read` scope only.

Scenarios: `/yandex-metrica-mcp:goal-conversions`, `/yandex-metrica-mcp:traffic-report`

Install:

```
/plugin install yandex-metrica-mcp --marketplace stufently/claude-plugins
```

or from a shell: `claude plugin marketplace add stufently/claude-plugins && claude plugin install yandex-metrica-mcp@stufently-mcp`.
