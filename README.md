# stufently MCP plugins for Claude Code

A Claude Code plugin marketplace for stufently's MCP servers. Each plugin installs the
server **and** ready-made scenarios (skills) built on its real tools, with safe
defaults: write switches off where the server has one, secrets only from your own
environment, image and package versions pinned.

```bash
claude plugin marketplace add stufently/claude-plugins
claude plugin install zabbix-ai-cli-mcp@stufently-mcp
```

Inside a session: `/plugin marketplace add stufently/claude-plugins`, then `/plugin`.

## Plugins

| Plugin | Scenarios | Runs | You provide (environment) |
| --- | --- | --- | --- |
| `zabbix-ai-cli-mcp` | `/zabbix-ai-cli-mcp:investigate-alert`, `/zabbix-ai-cli-mcp:whats-broken` | `docker` · `ghcr.io/stufently/zabbix-ai-cli-mcp:0.4.3` | `ZABBIX_AI_CLI_MCP_URL`, `ZABBIX_AI_CLI_MCP_TOKEN` |
| `google-webtools-mcp` | `/google-webtools-mcp:check-page-indexing`, `/google-webtools-mcp:weekly-seo-review` | `docker` · `ghcr.io/stufently/google-webtools-mcp:1.2.1` | `GOOGLE_APPLICATION_CREDENTIALS` (absolute path to a service-account key) |
| `bing-webmaster-ai-cli-mcp` | `/bing-webmaster-ai-cli-mcp:check-page-indexing`, `/bing-webmaster-ai-cli-mcp:traffic-review` | `uvx` · PyPI `bing-webmaster-ai-cli-mcp==0.1.2` | `BING_WM_API_KEY` |
| `yandex-metrica-mcp` | `/yandex-metrica-mcp:traffic-report`, `/yandex-metrica-mcp:goal-conversions` | `docker` · `ghcr.io/stufently/yandex-metrica-mcp:0.2.2` | `YANDEX_METRICA_TOKEN` |
| `aviasales-mcp` | `/aviasales-mcp:find-cheap-flight`, `/aviasales-mcp:where-to-fly` | `docker` · `ghcr.io/stufently/aviasales-mcp:0.5.1` | `AVIASALES_API_TOKEN`, optional `AVIASALES_MARKET` |
| `telegram-ai-cli-mcp` | `telegram-triage`, `telegram-write-by-plan` | the plugin in [its own repository](https://github.com/stufently/telegram-ai-cli-mcp), pinned by commit | `tg-ai` on `PATH`, an account signed in with `tg-ai account login` |

Set the variables in the shell that starts `claude`. The plugin configs pass them by
name (`docker run -e NAME`) or reference them (`${NAME}`); no value is stored in this
repository or in your settings.

Requirements: Docker for the `docker` plugins, [uv](https://docs.astral.sh/uv/) for
`uvx`. Scenarios are skills: Claude also picks them up on its own when a request
matches their description.

## Safe defaults

| Server | Default in this marketplace | Why |
| --- | --- | --- |
| Zabbix | `ZABBIX_AI_CLI_MCP_ALLOW_WRITE=false` (fixed in the plugin) | The server allows direct writes unless told otherwise; with this off `zabbix_write` is not offered. |
| Bing Webmaster | `BING_WM_ALLOW_WRITES=false` (fixed in the plugin) | Writes become `bing_plan_*` tools that only record a plan; you apply it with `bing-wm plan apply`. Spelled out because PyPI 0.1.2 still defaults to direct writes. |
| Telegram | `TGAI_PROFILE` default `readonly` (upstream plugin) | No tool sends anything in any profile; writes are plans applied from a terminal. |
| Google | no switch in the server | The server requests write scopes itself (`webmasters`, `analytics.edit`), so scopes do not limit it — the account's permissions do. Add the service account to Search Console as a **Restricted** user and to GA4 with the **Viewer** role. The scenarios never call the write tools (`add_property`, `submit_sitemap`, `ga4_create_property`, …). |
| Yandex Metrica | no switch in the server | Issue the OAuth token with the `metrika:read` scope only; counter create/delete then fail at the API. The scenarios never call those tools. |
| Aviasales | read-only API | Fares and lookups only; nothing books or pays. |

To change Zabbix or Bing directly from an agent, configure that server yourself
(see its README) instead of using this plugin.

## Not listed yet

- **yandex-mcp** (Search, Wordstat, Webmaster, Metrika, Direct). `@stufently/*` is not
  on npm yet and there is no container image, and the plugin in
  [stufently/yandex-mcp](https://github.com/stufently/yandex-mcp) starts its servers
  with `node` from the checkout, which has no `node_modules` when installed from
  GitHub. It will be added once the packages are published.

## Repository layout

```
.claude-plugin/marketplace.json   catalog (name: stufently-mcp)
plugins/<name>/
  .claude-plugin/plugin.json      manifest
  .mcp.json                       MCP server, pinned version, safe defaults
  skills/<scenario>/SKILL.md      scenarios
```

## Maintaining

- `claude plugin validate --strict .` and `claude plugin validate --strict plugins/<name>`
  before every push.
- Bumping a server version: change the image tag or package pin in `.mcp.json`, re-check
  the tool names the skills use against the server's `tools/list`, and bump `version`
  in that plugin's `plugin.json` — users stay on the old copy otherwise.
- `telegram-ai-cli-mcp` is pinned by `sha` in `marketplace.json`; move the pin to take a
  newer upstream plugin.

## License

MIT
