---
name: goal-conversions
description: Goal conversions in Yandex Metrica — list a counter's goals, then reaches and conversion rate per goal, by traffic source. Use for "how many leads/orders this week", "conversion by source", "is goal X firing".
argument-hint: "[counter ID] [goal name] [period]"
---

# Yandex Metrica goal conversions

Input: $ARGUMENTS — counter, optional goal name, period (default: last 7 days).

## Steps

1. Counter unknown → `yandex_metrica_list_counters`.
2. `yandex_metrica_list_goals` with `counter_id`. Match the goal the user named and
   note its ID. No goals → stop and say the counter has none configured.
3. `yandex_metrica_get_report` with `counter_ids` = [ID],
   `metrics` = [`ym:s:visits`, `ym:s:goal<GOAL_ID>reaches`, `ym:s:goal<GOAL_ID>conversionRate`]
   (up to 20 metrics, so several goals fit in one call),
   `dimensions` = [`ym:s:trafficSourceName`], `date1`/`date2`.
4. "Is goal X firing" → same metrics with `dimensions` = [`ym:s:date`]: days with zero
   reaches after non-zero ones point to a broken goal or tag.

## Answer

Per goal: reaches, conversion rate, best and worst source. Flag goals with zero reaches
in the period.

## Done when

Goal IDs come from `yandex_metrica_list_goals` and every figure from
`yandex_metrica_get_report` in this session.

## Limits

The server also exposes counter create/delete and Logs API export tools. This scenario
calls none of them. Use a token with the `metrika:read` scope only (see the plugin
README) so those tools cannot change anything.
