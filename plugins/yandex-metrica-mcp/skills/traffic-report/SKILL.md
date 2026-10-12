---
name: traffic-report
description: Yandex Metrica traffic report for a counter — visits, users, bounce rate and depth by day and by traffic source, optionally compared with the previous period. Use for "traffic for the last week", "where do visitors come from", "compare with last month" on a Metrica counter.
argument-hint: "[counter ID or site] [period]"
---

# Yandex Metrica traffic report

Input: $ARGUMENTS — a counter ID or site, and a period (default: last 7 days).

## Steps

1. **Counter.** No ID given → `yandex_metrica_list_counters` with `search` = the site
   name or domain. Several matches → ask which one.
2. **Totals by day.** `yandex_metrica_get_report` with `counter_ids` = [ID],
   `metrics` = [`ym:s:visits`, `ym:s:users`, `ym:s:bounceRate`, `ym:s:pageDepth`],
   `dimensions` = [`ym:s:date`], `date1`/`date2` for the period, `sort` = [`ym:s:date`].
3. **By source.** Same call with `dimensions` = [`ym:s:trafficSourceName`] and
   `sort` = [`-ym:s:visits`].
4. **Comparison, if asked.** Add `date1_b`/`date2_b` for the previous period of the
   same length; date1/date2 become segment A.

## Answer

Totals for the period (and the change vs the comparison period), a short day-by-day
line, and the source table. If the response says the data is sampled, say so.

## Done when

All numbers come from `yandex_metrica_get_report` in this session, with the counter ID
and exact dates stated.
