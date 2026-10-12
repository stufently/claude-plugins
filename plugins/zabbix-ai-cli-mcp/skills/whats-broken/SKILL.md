---
name: whats-broken
description: One-pass Zabbix overview — active problems by severity, hosts Zabbix cannot poll, and maintenance windows that may be hiding alerts. Use for "what is broken right now", "morning check", "is monitoring OK", or before a deploy.
argument-hint: "[host group | minimum severity]"
---

# What is broken right now

Optional scope: $ARGUMENTS (a host group or a minimum severity).

## Steps

1. `zabbix_problems` with `severity` = `average` (or what the user named), and
   `group` when given. Suppressed problems are included on purpose — keep them, and
   say which maintenance window hides each one.
2. `zabbix_unreachable` — hosts with at least one interface Zabbix cannot poll. Name
   the interface type (agent, SNMP, IPMI, JMX). Problems on that host that come from
   the same interface are likely downstream of it; problems from other interfaces or
   checks are not — keep them separate.
3. `zabbix_maintenance_list` with `active` = `true`. A window that is still open after
   the work it was for is a common reason alerts "stopped".
4. If the problem list was truncated, say so and offer to narrow by group or severity.

## Answer

A short table: severity, host, problem, age, acknowledged (yes/no), suppressed by.
Then unreachable hosts, then active maintenance windows. End with the one or two items
worth looking at first (highest severity × oldest × unacknowledged), and offer
`/zabbix-ai-cli-mcp:investigate-alert <event>` for any of them.

## Done when

Every row comes from a tool result in this session, and empty sections are reported as
"none" rather than left out.
