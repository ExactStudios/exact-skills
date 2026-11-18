# Plugin Check rule files

Rules the WordPress.org Plugins Team's [Plugin Check](https://wordpress.org/plugins/plugin-check/) plugin uses to judge its own warnings, rewritten in our words for this skill. Source: Plugin Check 2.2.0, `prompts/` (GPLv2 or later). Each file keeps the rules of one prompt and drops what only a one-off AI call needs (JSON fields, email tone); findings still follow the Finding contract in `../review-criteria.md`.

Plugin Check sends each warning to the prompt for its sniff code. Use the same table for warnings from `wp plugin check` and from the project's PHPCS run:

| Sniff code | Rule file | Category |
|------------|-----------|----------|
| `WordPress.Security.EscapeOutput`, `PluginCheck.CodeAnalysis.EscapeOutput` | `late-escaping.md` | WP.org Developer Expectations: Security; WP.com: Minimize security risks |
| `WordPress.Security.NonceVerification` | `nonce-verification.md` | WP.org Developer Expectations: Security; WP.com: Implement nonces |
| `WordPress.Security.ValidatedSanitizedInput` | `sanitization.md` | WP.org Developer Expectations: Security; WP.com: Data validation |
| `WordPress.DB.DirectDatabaseQuery`, `WordPress.DB.PreparedSQL` | `direct-db-queries.md` | WP.org Developer Expectations: Security; WP.com: Minimize security risks |
| `PluginCheck.CodeAnalysis.SettingSanitization` | `setting-sanitization.md` | WP.org Developer Expectations: Security; WP.com: Data validation |
| `PluginCheck.CodeAnalysis.Obfuscation` | `code-obfuscation.md` | WP.org 4: Human-readable code |
| `PluginCheck.CodeAnalysis.PluginUpdater` | `plugin-updater.md` | WP.org 8: Executable code from third parties |
| any other code | `generic.md` | the category the warning is about |

`naming.md` has no sniff code: apply it to the plugin's name, slug and description under WP.org 17, 9 and 12. Plugin Check's similar-name prompt (a web search for plugins and trademarks with similar names) is left out on purpose; this skill sends nothing about the plugin to outside services.

A warning these rules show to be a false positive is not a TODO row. List it in the final reply with the rule that dismissed it.

If Plugin Check is installed next to the audited plugin, its original prompts are in `wp-content/plugins/plugin-check/prompts/`. Read them, apply the stricter rule where they differ from these files, and report the difference.
