---
name: where-to-fly
description: Suggest where to fly from a city on a budget when the destination is open — cheapest destinations from the city, or anything under a price. Use for "where can I fly cheaply from X", "anywhere under 10000 rub", "weekend ideas from Bangkok".
argument-hint: "<from> [max price + currency]"
---

# Where to fly on a budget

Input: $ARGUMENTS — the departure city and optionally a budget.

## Steps

1. `lookup_cities` with `search` = the departure city → its IATA code.
2. With a budget → `search_by_price_range` with `origin`, `value_min` (0 unless
   given), `value_max`, and `currency` if the user named one. Leave `destination`
   unset. There is no date argument: each result carries its own `departure_at`.
3. Without a budget → `get_city_directions` with `origin`.
4. For the two or three destinations the user likes, price them properly with
   `get_prices_calendar` (cheapest day of a month) — see `find-cheap-flight`.

## Answer

A list of destinations with the cheapest fare, its dates and the booking link, sorted
by price. Say that fares are cached and per adult.

## Done when

Every destination and fare comes from this session's tool output.
