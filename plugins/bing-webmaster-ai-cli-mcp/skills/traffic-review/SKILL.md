---
name: traffic-review
description: Review a site's Bing search traffic — overall rank and clicks over time, top queries and top pages, and the queries behind one page. Use for "how is the site doing in Bing", "top Bing queries", "which pages get Bing traffic".
argument-hint: "[site URL] [page URL]"
---

# Bing traffic review

Input: $ARGUMENTS — the site (as registered in Bing) and optionally one page.

## Steps

1. Site unknown → `bing_sites_list` and pick it (ask if several fit).
2. `bing_traffic_rank` with `site_url` — clicks and impressions over time; compare
   the latest weeks with the ones before.
3. `bing_traffic_queries` with `site_url` — top queries by clicks.
4. `bing_traffic_pages` with `site_url` — top pages by clicks.
5. A page was named → `bing_traffic_page` with `site_url` and `page`; a query was
   named → `bing_query_page_stats` with `site_url` and `query`.

## Answer

Trend in one line (up/down/flat with the numbers), then top 10 queries and top 10
pages as tables (clicks, impressions, CTR, average position). Call out queries with
many impressions and low CTR — those are the title/description fixes.

## Done when

The trend, the query table and the page table all come from this session's tool output.
