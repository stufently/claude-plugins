---
name: check-page-indexing
description: Check whether Google has indexed a page and, if not, why — coverage verdict, canonical, robots, mobile and rich-result issues, plus whether the sitemap that should carry it is healthy. Use for "is this page in Google", "why is <url> not indexed", "Search Console says excluded".
argument-hint: "<page URL> [Search Console property]"
---

# Check a page's indexing in Google

Input: $ARGUMENTS — a page URL, optionally the Search Console property.

## Steps

1. **Find the property.** If the user did not give one, call `list_properties` and
   pick the property that covers the URL: `sc-domain:example.com` for a domain property,
   or the URL-prefix property such as `https://example.com/`. Use it exactly as listed
   as `siteUrl`. No matching property → stop and say the credentials cannot see this site.
2. **Inspect the URL.** `inspect_url` with `siteUrl` and `url`. Read: coverage state
   and verdict, Google-selected vs user-declared canonical, robots.txt state, indexing
   allowed (noindex), last crawl time, mobile usability, rich results.
3. **Several URLs** → `batch_inspect_urls` (up to 50) instead of looping step 2.
4. **The sitemap.** `list_sitemaps` with `siteUrl`; for the sitemap that should list
   this URL, `get_sitemap_details` with `siteUrl` and `feedpath` (the sitemap URL). Report errors, warnings and last download time.
5. **Optional, for a whole-site pass:** `check_indexing_issues` with `siteUrl` and
   `limit` (default 20) audits the top-traffic pages.

## Answer

First line: indexed / not indexed, and the reason in Google's own words (the coverage
state). Then the evidence fields, then the fix (remove noindex, fix canonical, add to
sitemap, etc.). Note that inspection reflects the last crawl, not the live page, and
that the URL Inspection API allows 2,000 inspections per property per day.

## Limits

The server can also submit/delete sitemaps and add/delete properties. This scenario
never calls them; if a fix needs one, describe it and ask before calling anything that
changes Search Console.

## Done when

The verdict, the canonical pair and the robots/noindex state are quoted from
`inspect_url`, and the sitemap status is stated.
