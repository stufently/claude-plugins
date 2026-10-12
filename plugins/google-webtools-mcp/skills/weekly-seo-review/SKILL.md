---
name: weekly-seo-review
description: Weekly Google Search Console review — traffic vs last week, growers and decliners, quick wins and CTR opportunities, with a short prioritized action list. Use for "weekly SEO report", "how did search do this week", "what should I fix first".
argument-hint: "[Search Console property]"
---

# Weekly SEO review

Input: $ARGUMENTS — the Search Console property; if missing, call `list_properties`
and ask which one when there are several.

## Steps

1. `weekly_seo_report` with `siteUrl`. This is the backbone: trend vs last week,
   growers, decliners, quick wins, sitemap health.
2. `find_declining_content` with `siteUrl` — pages losing clicks, to explain the
   decliners from step 1.
3. `find_ctr_opportunities` with `siteUrl` — pages ranking well but clicked below the
   benchmark for their position (title/description work).
4. Only if the user asks for a grade: `seo_health_check` with `siteUrl`.

## Answer

Five lines of headline numbers (clicks, impressions, CTR, position, change vs last
week), then at most five actions, each with the page or query it applies to and the
number that justifies it. Search Console data lags by about two days; say so if the
most recent days look low.

## Done when

Every number in the answer is from this week's tool output, and each action names a
concrete URL or query.
