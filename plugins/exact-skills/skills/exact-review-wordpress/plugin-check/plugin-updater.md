# Plugin updater

Source: Plugin Check 2.2.0, `prompts/ai-review-plugin-updater.md`, rewritten.

A plugin updater issue is a plugin that ships its own update mechanism instead of relying on WordPress.org updates. Plugins hosted on WordPress.org must not.

## Real issue

- A callback on `pre_set_site_transient_update_plugins` or `site_transient_update_plugins` that adds update data.
- A bundled update-checker library, such as `plugin-update-checker` (`YahnisElsts/plugin-update-checker`).
- A custom class that asks an external server whether an update is available.

## Not an issue

- License key validation that gates features rather than updates. That is a separate concern: WP.org 5 (trialware).
- Changes to the auto-update UI that only turn core's own auto-updates on or off.

## Note on `vendor/`

Plugin Check skips `vendor/` by default, so it may not report an updater there. That is the tool's default, not an exception in the guideline: this skill checks `vendor/` for WP.org 8.
