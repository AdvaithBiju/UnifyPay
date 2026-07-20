# UnifyPay SMS Parser Config — Changelog & Update Checklist

This file tracks every change to `sms_parser_config.json` and which app `versionCode` each
config release applies to. It is documentation only — nothing in the app reads this file.

## Mandatory checklist for every config update

Complete all of these before pushing any change to `sms_parser_config.json` on `main`:

1. **Validate the JSON.** Malformed JSON fails Gson parsing silently on-device — the fetch
   is caught, the whole update is skipped, no error is shown to you or to users.
2. **Confirm it will not crash or break older installs.**
   - Keyword lists / weights / thresholds that existing app code already understands →
     safe as-is, no gating needed.
   - A genuinely new schema field/section that only newer app code declares → harmless for
     old installs either way (Gson ignores unknown JSON keys), but bump
     `min_app_version_code` to the versionCode of the release that actually contains the
     matching Kotlin code, so older installs keep their current cached config instead of
     silently getting a config section their code doesn't know how to use.
   - Run the JVM test suite (`TransactionScorerTest`, `ScratchTest` — both load the real
     config file, not bare defaults) against the updated JSON locally before pushing.
3. **`latest_version_code` must match a versionCode that is actually live on Play Store**,
   not a local build or a work-in-progress one. Setting it ahead of a real release shows
   every eligible user an "Update" button that leads nowhere on Play Store — confirmed
   behavior: the update dialog in `AppStartupHost.kt` fires purely from
   `latest_version_code > installed versionCode`, independent of `min_app_version_code`.
4. **`update_message` must be the real, accurate, user-facing message** — never leave
   placeholder or test text live. If testing the update-dialog mechanism itself, use an
   obviously-labeled test string temporarily and revert it before ending the session.
5. **After pushing, verify the raw file on GitHub** actually reflects the intended content
   (`raw.githubusercontent.com/AdvaithBiju/UnifyPay/main/sms_parser_config.json`) — the CDN
   can cache briefly, so re-check if a device doesn't pick it up within a few minutes.

## Version history

| Config `version` | Date | App versionCode(s) | Commit | What changed |
|---|---|---|---|---|
| 1 | 2026-06-27 | 10 | `ab89b58` | Initial remote config: `routing_rules`, keyword-blocklist `pre_filter`, `reference_patterns`, `merchant_categories`. |
| 2 | 2026-06-27 | 10 | `f7ebe26` | Added `latest_version_code` / `update_message` — enabled the in-app update-nag dialog. |
| 3 | 2026-07-08 | 13–14 | `6d31e0a` | Fixed false positives: postpaid/prepaid substring match on "paid", demat/securities balance SMS, shortlink promo SMS (added `shortlink_domains`, expanded `promo_keywords`). |
| — | 2026-07-13 | 13→14 | `1e0410f`, `e9ffd12` | versionCode-only correction (13 was rejected by Play Console, corrected to 14). No schema change. |
| 4 | 2026-07-20 | 15+ (`min_app_version_code: 15`) | `0390dce` | Replaced the keyword-blocklist detection with a positive-evidence weighted-scoring model (`scoring` + `sender_classification` sections). Fixed a live bug: `hasRefPattern` used substring `.contains()` instead of word-boundary matching, letting "refer"/"referral" falsely satisfy the reference-number check. Added `outcome_reject_patterns` for declined/failed/reversed messages. Gated to versionCode 15+; installs on 14 or below are unaffected. |
| 4 (metadata only) | 2026-07-20 | n/a | `b3cd6c4` → reverted | Briefly set `latest_version_code: 16` with a placeholder test message to verify the update-dialog mechanism end-to-end on a real device. Reverted to real values immediately after — see checklist item 4 above, added because of exactly this. |
