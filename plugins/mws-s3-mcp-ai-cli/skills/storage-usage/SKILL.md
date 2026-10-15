---
name: storage-usage
description: Show where the space goes in a MWS Cloud Platform S3 bucket — usage by top-level prefix, the largest objects and forgotten unfinished multipart uploads — with the cleanup commands a person can run. Use for "what takes space in the bucket", "why did the S3 bill grow", "find big files in S3".
---

# Where the space goes in MWS Cloud Platform S3

Input: a bucket and optionally a prefix. Use the `mws-s3` server's tools; they scan a bounded number of keys (default 10 000), so a large bucket gives a partial picture — say so and offer to continue with the returned `continuation_token`.

## Steps

1. **By prefix.** `s3_prefix_usage` with `bucket` (and `prefix` to drill down). Rank prefixes by bytes and object count.
2. **Largest objects.** `s3_largest_objects` with `bucket`, `limit` = 20. Point out duplicates by name pattern (several full dumps of the same database, old archives next to new ones).
3. **Unfinished uploads.** `s3_unfinished_uploads` with `bucket` (default: older than 24 hours). Their parts are billed but invisible in a normal listing.
4. **Older versions.** If `s3_get_bucket_config` shows versioning `Enabled` or `Suspended`, note that deleted or overwritten objects still use space as old versions; `s3_list_object_versions` on the heaviest prefix shows how many.

## Answer

A short table: prefix, size, objects, share of the scanned total; then the five largest objects and the unfinished uploads. End with what a person could clean up and the commands to review first, for example `mws-s3 ls <bucket>/<prefix> --json` and `mws-s3 rm <bucket>/<key>` (asks for confirmation). This plugin's server is read-only and never deletes or aborts anything itself.
