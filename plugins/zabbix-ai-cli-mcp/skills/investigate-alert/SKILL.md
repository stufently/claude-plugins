---
name: investigate-alert
description: Investigate one Zabbix alert end to end — from a pasted notification, an event ID or a host name to the cause, the metric behind it and whether the notification reached anyone. Use when the user pastes a Zabbix alert, asks "what is this alert", "why is <host> alerting" or "why didn't I get notified".
argument-hint: "[pasted alert text | event ID | host]"
---

# Investigate a Zabbix alert

Input: $ARGUMENTS (a pasted notification, an event ID, or a host name). Gather facts
with the `zabbix` server's tools first, then explain. Do not guess identifiers.

## Steps

1. **Get identifiers.**
   - Pasted notification text → `zabbix_resolve` with `text` = the whole paste.
     It returns the event, host and trigger IDs.
   - Bare event ID → go to step 2.
   - Only a host name or fragment → `zabbix_hosts` with `search`, then
     `zabbix_problems` with `host` to pick the event. Several matches → ask which one.
2. **The problem itself.** `zabbix_problem` with `event`. Note host, severity, age,
   acknowledgement, and whether a maintenance window suppresses it.
3. **The host around it.** `zabbix_host_investigate` with `host`. Read in this order:
   agent unavailable → everything else is downstream; in maintenance → alerts are
   suppressed; silent or unsupported items → a monitoring fault, not necessarily a host
   fault; then the other active problems and the last 24h of events (repeated
   problem/resolve pairs for one trigger mean flapping).
4. **The value behind a threshold trigger.** `zabbix_metrics_history` with `host`,
   `search` = the item name or key fragment from the trigger, `last` = `24h`
   (or `7d` for slow trends). Quote min / avg / max from the summary.
5. **Delivery, only if the user asks about notifications** (or the problem is old and
   unacknowledged): `zabbix_alert_why` with `event`. It walks suppression, actions,
   media types and each recipient's severity filter and active period.

## Answer

Lead with one sentence: what is wrong, on which host, since when. Then the evidence
(tool, field, value) and one concrete next step. Mark anything you inferred rather than
read as an inference.

## Limits

This plugin starts the server with `ZABBIX_AI_CLI_MCP_ALLOW_WRITE=false`: there is no
tool that acknowledges, closes or silences anything. If the user wants that, say so and
point them to the `zabbix-ai-cli-mcp` CLI (`plan` / `approve`) on their own machine.

## Done when

The answer names the event ID, the host, the triggering value or the reason there is
none, and — when asked — the link in the delivery chain that dropped the notification.
