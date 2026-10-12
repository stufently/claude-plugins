---
name: find-cheap-flight
description: Find the cheapest flight for a route — cheapest day in a month, cheaper days around chosen dates, nearby airports — with booking links for the whole party. Use for "cheapest flight from X to Y", "when is it cheapest to fly to Y in March", "is it cheaper a day earlier".
argument-hint: "<from> <to> [date or month] [passengers]"
---

# Find a cheap flight

Input: $ARGUMENTS — origin, destination, a date (YYYY-MM-DD) or month (YYYY-MM), and
optionally passengers and class.

## Steps

1. **IATA codes.** For each place that is not already a code, `lookup_cities` with
   `search` = the name (pass `locale` = `ru` for a Cyrillic name). Several hits → ask.
   A town with no airport of its own → `find_nearest_airports` with `near`.
2. **Pick the call by what the user fixed:**
   - only a month, or "when is cheapest" → `get_prices_calendar` with `origin`,
     `destination`, `departure_at` = `YYYY-MM` (and `return_at` for a round trip);
   - exact dates and "could another day be cheaper" → `get_flexible_date_prices`
     with `depart_date` (and `return_date`);
   - exact date or month and they want concrete tickets → `search_flights` with
     `departure_at`, `return_at` or `one_way` = `true`, `direct` if they want non-stop.
   Pass `adults`/`children`/`infants`/`trip_class` when given: prices stay per adult,
   but the booking link opens with the whole party.
3. **Nearby airports**, if the user is flexible or results are thin:
   `get_alternative_directions` with `origin` and `destination`.
4. Nothing found → say the cache has no fares for that route and dates; do not invent
   prices. Offer the month view or nearby airports.

## Answer

The three cheapest options: price per adult with currency, dates, stops and
`duration_total`, and the `booking_link`. State plainly that these are cached fares
from recent searches, per adult, and the final price is on the booking page.

## Done when

Every price and link comes from a tool result in this session.
