# bing-webmaster-ai-cli-mcp

Claude Code plugin for [bing-webmaster-ai-cli-mcp](https://github.com/stufently/bing-webmaster-ai-cli-mcp). See the [marketplace README](../../README.md) for install, environment variables and safe defaults.

## What it runs and where it connects

The plugin starts the PyPI package `bing-webmaster-ai-cli-mcp==0.1.2` with `uvx`. The server talks only to the Bing Webmaster API with your `BING_WM_API_KEY`. Credentials come from environment variables in the shell that starts Claude Code; the plugin stores none. The server starts with `BING_WM_ALLOW_WRITES=false`: changes are recorded as plans that you apply yourself with `bing-wm plan apply`.

Scenarios: `/bing-webmaster-ai-cli-mcp:check-page-indexing`, `/bing-webmaster-ai-cli-mcp:traffic-review`

Install:

```
/plugin install bing-webmaster-ai-cli-mcp --marketplace stufently/claude-plugins
```

or from a shell: `claude plugin marketplace add stufently/claude-plugins && claude plugin install bing-webmaster-ai-cli-mcp@stufently-mcp`.
