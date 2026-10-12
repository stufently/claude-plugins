# zabbix-ai-cli-mcp

Claude Code plugin for [zabbix-ai-cli-mcp](https://github.com/stufently/zabbix-ai-cli-mcp). See the [marketplace README](../../README.md) for install, environment variables and safe defaults.

## What it runs and where it connects

The plugin starts the container image `ghcr.io/stufently/zabbix-ai-cli-mcp:0.4.3` with `docker run`. The server talks only to your Zabbix server at `ZABBIX_AI_CLI_MCP_URL`, authenticated with `ZABBIX_AI_CLI_MCP_TOKEN`. Credentials come from environment variables in the shell that starts Claude Code; the plugin stores none. The server starts with `ZABBIX_AI_CLI_MCP_ALLOW_WRITE=false`, so it offers no tool that changes Zabbix.

Scenarios: `/zabbix-ai-cli-mcp:investigate-alert`, `/zabbix-ai-cli-mcp:whats-broken`

Install:

```
/plugin install zabbix-ai-cli-mcp --marketplace stufently/claude-plugins
```

or from a shell: `claude plugin marketplace add stufently/claude-plugins && claude plugin install zabbix-ai-cli-mcp@stufently-mcp`.
