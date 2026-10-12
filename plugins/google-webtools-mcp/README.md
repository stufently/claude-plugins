# google-webtools-mcp

Claude Code plugin for [google-webtools-mcp](https://github.com/stufently/google-webtools-mcp). See the [marketplace README](../../README.md) for install, environment variables and safe defaults.

## What it runs and where it connects

The plugin starts the container image `ghcr.io/stufently/google-webtools-mcp:1.2.1` with `docker run`, mounting the key file named by `GOOGLE_APPLICATION_CREDENTIALS` read-only. The server talks only to Google's Search Console, Analytics and Site Verification APIs. Credentials come from environment variables in the shell that starts Claude Code; the plugin stores none. The server also has write tools; the scenarios never call them. Give the service account Restricted access in Search Console and the Viewer role in GA4.

Scenarios: `/google-webtools-mcp:check-page-indexing`, `/google-webtools-mcp:weekly-seo-review`

Install:

```
/plugin install google-webtools-mcp --marketplace stufently/claude-plugins
```

or from a shell: `claude plugin marketplace add stufently/claude-plugins && claude plugin install google-webtools-mcp@stufently-mcp`.
