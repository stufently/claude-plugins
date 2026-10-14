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
While this repository is private, `marketplace add` clones it with the git credentials
already on your machine (`gh auth login` or an SSH key that works without a prompt).

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
  README.md, LICENSE              per-plugin docs (the directory ships only this folder)
  skills/<scenario>/SKILL.md      scenarios
```

## Maintaining

- `claude plugin validate --strict .` and `claude plugin validate --strict plugins/<name>`
  before every push.
- Bumping a server version: change the image tag and digest (`docker buildx imagetools inspect`)
  or the package pin in `.mcp.json` and the plugin README, re-check
  the tool names the skills use against the server's `tools/list`, and bump `version`
  in that plugin's `plugin.json` — users stay on the old copy otherwise.
- `telegram-ai-cli-mcp` is pinned by `sha` in `marketplace.json`; move the pin to take a
  newer upstream plugin.

## Anthropic directory

See [SUBMISSION.md](SUBMISSION.md) for what to enter in the developer portal per plugin.

## License

MIT
