# google-webtools-mcp

Check why a page is not in Google and get a weekly Search Console review with a short action list, through the google-webtools MCP server (Search Console and GA4).

This is a community plugin for the open-source MCP server [google-webtools-mcp](https://github.com/stufently/google-webtools-mcp). It is not made or endorsed by the vendor of the service it connects to.

## Scenarios

`/google-webtools-mcp:check-page-indexing`, `/google-webtools-mcp:weekly-seo-review`. They are skills: Claude also picks them up on its own when a request matches.

## Configuration

Claude Code asks for these values when you enable the plugin; change them later in `/config` (secret values are not shown there; re-enter them with `claude plugin configure google-webtools-mcp`). Values marked secret are stored in the system credential store, not in `settings.json`, and reach the server only through its environment.

| Setting | Key | Secret | What it is |
| --- | --- | --- | --- |
| Service account key file | `credentials_file` | no (a path; the key stays in your file) | Absolute path to a Google service-account JSON key. |

Requires Docker.

## What it runs and where data goes

The plugin starts the container image `ghcr.io/stufently/google-webtools-mcp:1.2.1`, pinned by digest, with `docker run -i --rm`, mounting the service-account key file you choose read-only at `/creds/sa.json`. Docker pulls the image from GitHub Container Registry (`ghcr.io`) the first time. The server talks only to Google's Search Console, Analytics (GA4) and Site Verification APIs (`*.googleapis.com`, including the OAuth token endpoint), authenticated as your service account. It sends nothing anywhere else, and the plugin has no hooks or telemetry.

## Safe defaults

The server has no read-only switch and requests write scopes itself (`webmasters`, `analytics.edit`), so the account's permissions are what limit it: add the service account to Search Console as a **Restricted** user and to GA4 with the **Viewer** role. The scenarios never call the write tools (`add_property`, `submit_sitemap`, `ga4_create_property`, and others).

## Install

```
/plugin marketplace add stufently/claude-plugins
/plugin install google-webtools-mcp@stufently-mcp
```

## License

MIT, see [LICENSE](LICENSE).
