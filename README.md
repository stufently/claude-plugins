# stufently MCP plugins for Claude Code

A Claude Code plugin marketplace for stufently's MCP servers. Each plugin installs the
server **and** ready-made scenarios (skills) built on its real tools, with safe
defaults: write switches off where the server has one, secrets asked for on enable and
kept in the system credential store, image and package versions pinned.

```bash
claude plugin marketplace add stufently/claude-plugins
claude plugin install zabbix-ai-cli-mcp@stufently-mcp
```

Inside a session: `/plugin marketplace add stufently/claude-plugins`, then `/plugin`.
The repository is public, so `marketplace add` needs no GitHub credentials.

### Settings and secrets (`userConfig`)

Each plugin declares its settings in `userConfig`; the table below lists them. Set them
in one of these ways:

- **In a session**: `/plugin`, **Installed** tab, the plugin, **Configure options**.
  Claude Code also asks for them when you enable the plugin there; secret fields are
  masked.
- **From your shell**, reading the secret from a hidden prompt so it never appears on
  the command line or in shell history (needs `jq`):
  ```bash
  read -rsp 'Zabbix API token: ' ZBX_TOKEN; echo
  jq -cn --arg url https://zabbix.example.com --arg token "$ZBX_TOKEN" \
    '{zabbix_url: $url, zabbix_token: $token}' \
    | claude plugin configure zabbix-ai-cli-mcp@stufently-mcp --values-stdin
  unset ZBX_TOKEN
  ```
  `claude plugin configure <plugin>@stufently-mcp` with no flags shows which options are
  still unset. Options left out of the JSON keep their values.
- `claude plugin install <plugin>@stufently-mcp --config KEY=VALUE` also works, but use it
  only for non-secret keys: the value stays in your shell history.

Until the required settings are set, the plugin installs but its MCP server has no
credentials. Restart Claude Code (or `/reload-plugins`) after changing settings.
Non-secret values appear as rows in `/config`; secret ones do not, so change a token by
configuring it again.

## Plugins

| Plugin | Scenarios | Runs | Asked for on enable (`userConfig`) |
| --- | --- | --- | --- |
| `zabbix-ai-cli-mcp` | `/zabbix-ai-cli-mcp:investigate-alert`, `/zabbix-ai-cli-mcp:whats-broken` | `docker` · `ghcr.io/stufently/zabbix-ai-cli-mcp:0.4.3` (pinned by digest) | Zabbix URL; API token (secret) |
| `google-webtools-mcp` | `/google-webtools-mcp:check-page-indexing`, `/google-webtools-mcp:weekly-seo-review` | `docker` · `ghcr.io/stufently/google-webtools-mcp:1.2.1` (pinned by digest) | path to a service-account key file |
| `bing-webmaster-ai-cli-mcp` | `/bing-webmaster-ai-cli-mcp:check-page-indexing`, `/bing-webmaster-ai-cli-mcp:traffic-review` | `uvx` · PyPI `bing-webmaster-ai-cli-mcp==0.1.3` | API key (secret) |
| `yandex-metrica-mcp` | `/yandex-metrica-mcp:traffic-report`, `/yandex-metrica-mcp:goal-conversions` | `docker` · `ghcr.io/stufently/yandex-metrica-mcp:0.2.2` (pinned by digest) | OAuth token (secret) |
| `aviasales-mcp` | `/aviasales-mcp:find-cheap-flight`, `/aviasales-mcp:where-to-fly` | `docker` · `ghcr.io/stufently/aviasales-mcp:0.5.2` (pinned by digest) | Travelpayouts API token (secret); market, default `ru` |
| `telegram-ai-cli-mcp` | `telegram-triage`, `telegram-write-by-plan` | the plugin in [its own repository](https://github.com/stufently/telegram-ai-cli-mcp), pinned by commit (0.1.2) | nothing: `tg-ai` on `PATH`, an account signed in with `tg-ai account login` |

Secret values go to the system credential store, the rest to `pluginConfigs` in your `settings.json`;
the plugin passes them to the server through its environment (`${user_config.KEY}`
in `.mcp.json`). Nothing is read from your shell environment, and no value is stored
in this repository.

Requirements: Docker for the `docker` plugins, [uv](https://docs.astral.sh/uv/) for
`uvx`. Scenarios are skills: Claude also picks them up on its own when a request
matches their description. Each plugin folder has its own README with what it runs,
where data goes and its settings.

## Safe defaults

| Server | Default in this marketplace | Why |
| --- | --- | --- |
| Zabbix | `ZABBIX_AI_CLI_MCP_ALLOW_WRITE=false` (fixed in the plugin) | The server allows direct writes unless told otherwise; with this off `zabbix_write` is not offered. |
| Bing Webmaster | `BING_WM_ALLOW_WRITES=false` (fixed in the plugin) | Writes become `bing_plan_*` tools that only record a plan; you apply it with `bing-wm plan apply`. Also the server default since 0.1.3; kept explicit. |
| Telegram | `TGAI_PROFILE=readonly` (fixed in the upstream plugin) | No tool sends anything in any profile; writes are plans applied from a terminal. |
| Google | no switch in the server | The server requests write scopes itself (`webmasters`, `analytics.edit`), so scopes do not limit it — the account's permissions do. Add the service account to Search Console as a **Restricted** user and to GA4 with the **Viewer** role. The scenarios never call the write tools (`add_property`, `submit_sitemap`, `ga4_create_property`, …). |
| Yandex Metrica | no switch in the server | Issue the OAuth token with the `metrika:read` scope only; counter create/delete then fail at the API. The scenarios never call those tools. |
| Aviasales | read-only API | Fares and lookups only; nothing books or pays. |

To change Zabbix or Bing directly from an agent, configure that server yourself
(see its README) instead of using this plugin.

## Not listed yet

- **yandex-mcp** (Search, Wordstat, Webmaster, Metrika, Direct). All five
  `@stufently/yandex-*-mcp` packages are on npm (2.2.0, 2026-10-15), so a plugin here
  can run them with `npx -y`. Not added yet: the plugin in
  [stufently/yandex-mcp](https://github.com/stufently/yandex-mcp) still starts its
  servers with `node` from the checkout, which has no `node_modules` when installed
  from GitHub. Until then, use the npm packages directly (see that repository's README).

## Cursor

The same repository is a Cursor plugin marketplace (`.cursor-plugin/marketplace.json`,
format from [cursor.com/docs/reference/plugins](https://cursor.com/docs/reference/plugins)).
Each Cursor plugin bundles the MCP server, the scenarios as skills, a slash command per
scenario and one rule with the server's safety limits. The five plugins above reuse their
`skills/` folder and pin the same image or package; the seven S3 plugins exist only for
Cursor so far.

| Plugin | Commands (scenario) | Runs | Variables |
| --- | --- | --- | --- |
| `zabbix-ai-cli-mcp` | `/zabbix-investigate-alert`, `/zabbix-whats-broken` | `docker`, writes off | `ZABBIX_URL`, `ZABBIX_TOKEN` |
| `google-webtools-mcp` | `/gsc-check-page-indexing`, `/gsc-weekly-seo-review` | `docker` | `GOOGLE_CREDENTIALS_FILE` (path to the key) |
| `bing-webmaster-ai-cli-mcp` | `/bing-check-page-indexing`, `/bing-traffic-review` | `uvx`, writes as plans | `BING_WM_API_KEY` |
| `yandex-metrica-mcp` | `/metrica-traffic-report`, `/metrica-goal-conversions` | `docker` | `YANDEX_METRICA_TOKEN` |
| `aviasales-mcp` | `/aviasales-find-cheap-flight`, `/aviasales-where-to-fly` | `docker` | `AVIASALES_API_TOKEN`, `AVIASALES_MARKET` |
| `<provider>-s3-mcp-ai-cli` × 7 (Selectel, Yandex Cloud, VK Cloud, Timeweb, MWS, Cloud.ru, Beget) | `/s3-<provider>-restore`, `/s3-<provider>-usage` | `docker` · `ghcr.io/stufently/<provider>-s3-mcp-ai-cli:0.1.0`, read-only | `<PROVIDER>_S3_ACCESS_KEY_ID`, `<PROVIDER>_S3_SECRET_ACCESS_KEY`; `<PROVIDER>_S3_POOL` (defaults to the provider pool) |

Secrets are plugin `variables`: `cursor/mcp.json` holds only `${VAR}` placeholders, and
the values are set in Cursor under **Plugins → Configure**. Until the repository is in a
marketplace, a team admin can add it under **Dashboard → Plugins & MCPs → Team
Marketplaces → Import from Repo** (Teams/Enterprise plans). Single servers without the
scenarios install with the **Add to Cursor** button in each server's README.

Telegram (`telegram-ai-cli-mcp`) and `yandex-mcp` are not in the Cursor marketplace yet:
their plugins live in their own repositories.

## Repository layout

```
.claude-plugin/marketplace.json   catalog (name: stufently-mcp)
plugins/<name>/
  .claude-plugin/plugin.json      manifest
  .mcp.json                       MCP server, pinned version, safe defaults
  README.md, LICENSE              per-plugin docs (the directory ships only this folder)
  skills/<scenario>/SKILL.md      scenarios (shared with Cursor)
  .cursor-plugin/plugin.json      Cursor manifest (paths below, variables)
  cursor/mcp.json                 Cursor MCP config with ${VAR} placeholders
  cursor/rules/*.mdc              Cursor rule: the server's safety limits
  cursor/commands/*.md            Cursor slash commands, one per scenario
.cursor-plugin/marketplace.json   Cursor catalog (name: stufently-mcp)
```

Cursor-only files sit under `cursor/` so Claude Code, which discovers `commands/` in a
plugin, does not pick them up.

## Maintaining

- `claude plugin validate --strict .` and `claude plugin validate --strict plugins/<name>`
  before every push.
- Bumping a server version: change the image tag and digest (`docker buildx imagetools inspect`)
  or the package pin in `.mcp.json` and the plugin README, re-check
  the tool names the skills use against the server's `tools/list`, and bump `version`
  in that plugin's `plugin.json` — users stay on the old copy otherwise.
- Cursor: `cursor/mcp.json` must pin the same image/package as `.mcp.json`; bump `version`
  in `.cursor-plugin/plugin.json` too. Validate with Cursor's scripts from
  [cursor/plugins](https://github.com/cursor/plugins) (`scripts/validate-plugins.mjs`
  and the schemas) and [cursor/plugin-template](https://github.com/cursor/plugin-template)
  (`scripts/validate-template.mjs`), run from the repository root.
- `telegram-ai-cli-mcp` is pinned by `sha` in `marketplace.json`; move the pin to take a
  newer upstream plugin.

## Anthropic directory

See [SUBMISSION.md](SUBMISSION.md) for what to enter in the developer portal per plugin,
and for the Cursor Marketplace submission.

## License

MIT
