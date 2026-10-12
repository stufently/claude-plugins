---
name: check-page-indexing
description: Check whether Bing has crawled and indexed a page, what happened on the last fetch, and whether crawl issues or blocks explain a missing page; optionally prepare a reviewed plan to submit it. Use for "is this page in Bing", "why doesn't Bing show <url>", "did Bing crawl it".
argument-hint: "<page URL> [site URL as registered in Bing]"
---

# Check a page's indexing in Bing

Input: $ARGUMENTS — a page URL, optionally the site as registered in Bing Webmaster.

## Steps

1. **Find the site.** If not given, `bing_sites_list` and pick the verified site
   that covers the URL; use it as `site_url`. Not listed or not verified → stop and say so.
2. **Crawl record.** `bing_url_info` with `site_url` and `url`: last crawl date,
   HTTP status, whether it is in the index, discovery date.
3. **Last fetch detail.** `bing_fetched_url_details` with `site_url` and `url`, if
   Bing fetched it recently (otherwise say there is no recent fetch).
4. **Site-wide causes.** `bing_crawl_issues` with `site_url` (look for this URL or its
   path), and `bing_blocked_urls` with `site_url`. Only a block with RequestType 1
   (FullRemoval) hides the page from results; RequestType 0 (CacheOnly) removes only
   the cached copy and does not explain a missing page.
5. **Sitemap.** `bing_sitemaps` with `site_url`; for the sitemap that should list the
   page, `bing_sitemap_details` with `site_url` and `feed_url`.
6. **Submitting (only if the user asks).** `bing_submission_quota` with `site_url`,
   then `bing_plan_submit_url` with `site_url` and `url`. This only records a plan —
   nothing is sent to Bing. Give the user the `bing-wm plan apply <plan_id>` command
   from the result to run themselves. Never say the URL was submitted.

## Answer

First line: in the index or not, last crawl date and HTTP status. Then the blocking
cause if one was found (crawl issue, block request, missing from sitemap), and the
next step.

## Limits

This plugin starts the server with `BING_WM_ALLOW_WRITES=false`, so the only write
path is a plan a person applies with `bing-wm` at a terminal.

## Done when

The crawl/index state is quoted from `bing_url_info`, and blocks and crawl issues for
the URL are reported as found, or as "nothing returned" — Bing can answer with an empty
list while issues exist (check the result's `empty_response` flag), so an empty answer
is not proof that there is no problem.
