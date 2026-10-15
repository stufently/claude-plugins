---
name: restore-from-backup
description: Find the right backup copy in Yandex Cloud S3 and hand it over for restore — locate the object by bucket, prefix, name or date, confirm size and age, check older versions when the bucket keeps them, then give a short-lived download link or the exact CLI command. Use for "restore a file from S3", "find yesterday's backup", "get the previous version of this object".
---

# Restore a file from a Yandex Cloud S3 backup

Input: what to restore (bucket, prefix, file name, approximate date). Use the `yandex-s3` server's tools. The plugin runs it read-only: it never overwrites, deletes or moves anything.

## Steps

1. **Bucket.** If the bucket is not given, `s3_list_buckets` and ask which one holds the backups.
2. **Candidates.** `s3_list_objects` with `bucket` and the narrowest `prefix` you can (follow `continuation_token` while the listing is cut). Pick candidates by name and `LastModified`; several plausible copies → show them and ask.
3. **Confirm the copy.** `s3_head_object` on the chosen key: size, last-modified, ETag, content type. A size of zero or far below its neighbours means a failed backup — say so and offer the previous copy.
4. **Older versions, if needed.** `s3_get_bucket_config` shows the versioning status. If it is `Enabled` or `Suspended` (versions kept from before suspension stay readable) and the user needs an earlier state, `s3_list_object_versions` with the key as `prefix` and pick the `version_id` by date. Skip this step when the tool is not offered or versioning was never on.
5. **Peek, only for text.** For configs or dumps in text form, `s3_preview_object` (first 64 KiB) to confirm it is the right file. Never pull large binaries into the chat.
6. **Hand over.**
   - `s3_presign_get` with `bucket`, `key`, optional `version_id` and `expires_seconds` = 900 gives a download link. Tell the user it works for anyone who has it until it expires.
   - Otherwise, or for whole folders, give the CLI command to run on the target machine: `yandex-s3 cp s3://<bucket>/<key> ./<file>` (add `--recursive` for a prefix). Downloads are atomic; an existing local file is never silently replaced.

## Done when

The answer names the bucket, key, size, last-modified time (and `version_id` when used), and gives one way to download it. After the user restores, suggest comparing the size (and checksum, if they keep one) with the object metadata.
