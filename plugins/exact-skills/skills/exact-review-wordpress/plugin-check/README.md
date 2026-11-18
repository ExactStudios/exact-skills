# Plugin Check rule files

Rules the WordPress.org Plugins Team's [Plugin Check](https://wordpress.org/plugins/plugin-check/) plugin uses to judge its own warnings, rewritten in our words for this skill. Source: Plugin Check 2.2.0, `prompts/` (GPLv2 or later). Each file keeps the rules of one prompt and drops what only a one-off AI call needs (JSON fields, email tone); findings still follow the Finding contract in `../review-criteria.md`.

Plugin Check sends each warning to the prompt whose code its full sniff code starts with (`WordPress.Security.EscapeOutput.OutputNotEscaped` starts with `WordPress.Security.EscapeOutput`). Use the same table, matched by prefix, for warnings from `wp plugin check` and from the project's PHPCS run:

| Sniff code starts with | Rule file | Category |
|------------|-----------|----------|
| `WordPress.Security.EscapeOutput`, `PluginCheck.CodeAnalysis.EscapeOutput` | `late-escaping.md` | WP.org Developer Expectations: Security; WP.com: Minimize security risks |
| `WordPress.Security.NonceVerification` | `nonce-verification.md` | WP.org Developer Expectations: Security; WP.com: Implement nonces |
| `WordPress.Security.ValidatedSanitizedInput` | `sanitization.md` | WP.org Developer Expectations: Security; WP.com: Data validation |
| `WordPress.DB.DirectDatabaseQuery`, `WordPress.DB.PreparedSQL` | `direct-db-queries.md` | WP.org Developer Expectations: Security; WP.com: Minimize security risks |
| `PluginCheck.CodeAnalysis.SettingSanitization` | `setting-sanitization.md` | WP.org Developer Expectations: Security; WP.com: Data validation |
| `PluginCheck.CodeAnalysis.Obfuscation` | `code-obfuscation.md` | WP.org 4: Human-readable code |
| `PluginCheck.CodeAnalysis.PluginUpdater` | `plugin-updater.md` | WP.org 8: Executable code from third parties |
| any other code | `generic.md` | the category the warning is about |

`naming.md` has no sniff code; WP.org 17, 9 and 12 in the criteria point to it.
