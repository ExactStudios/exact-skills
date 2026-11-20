---
name: exact-review-wordpress
description: Use when the user asks for a read-only audit of one WordPress plugin folder against the WordPress.org plugin guidelines, the WordPress.com marketplace plugin guidelines, or both, with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file> [org|com|both]"
disable-model-invocation: true
metadata:
  author: Shakeeb Ahmed
  tags: [code-review, wordpress, plugin-guidelines, audit]
---

# Exact Review WordPress

Audit one WordPress plugin folder against the WordPress.org Detailed Plugin Guidelines and the WordPress.com plugin developer guidelines, and record verified, actionable findings in a TODO file, each under the guideline it breaks. Analysis only.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with this profile.

## Profile

- **Arguments**: `$ARGUMENTS`. After `FOLDER` and `TODO_FILE`, an optional third word `TARGET`: `org` (WordPress.org directory), `com` (WordPress.com marketplace) or `both`. Default `both`; don't ask for it. Any other value: stop and ask.
- **Criteria**: `review-criteria.md` in this skill's folder
- **Source**: every file that ships in the plugin package: `.php`, `.js`, `.css`, `readme.txt`, license files, `block.json`, translation files (`.pot`, `.po`), images, fonts, and `composer.json`, `composer.lock`, `package.json` and `package-lock.json`. Include `vendor/`, `build/`, `dist/` and minified files; the guidelines on licenses, readable code and default libraries live there. In those, check only WP.org 1, 2, 4, 7, 8 and 13, and give each bundled library one slice whatever its size, reading its license, manifest and entry files rather than every line.
- **Exclusions**: `node_modules`, `.git`, and files the deploy config keeps out of the shipped package (`.distignore`, `export-ignore` in `.gitattributes`, the build or zip script). Still read excluded build sources and config as evidence for WP.org 4.
- **Project files**: the main plugin file header, `readme.txt`, `LICENSE` or `COPYING`, `composer.json` and `composer.lock`, `package.json` and its lockfile, the build config and the deploy config, to learn the slug (the folder name), the main file, the text domain, the version in each place it is declared, "Requires at least", "Requires PHP", the license, the bundled libraries with their licenses and versions, what ships, and whether readable source ships or the readme links it.
- **Extra prep**:
  - Resolve `TARGET` and apply only its categories from the Criteria.
  - Decide whether the plugin depends on WooCommerce (a `Requires Plugins` header naming `woocommerce`, WooCommerce class or function checks); the WooCommerce-only rules apply only then.
  - Decide whether it is a Block Plugin (it exists only to distribute one block); the Block Directory rules apply only then.
  - Read `plugin-check/README.md` in this skill's folder for the sniff-code table, and resolve the folder's absolute path for sub-agents. If Plugin Check is installed in the same WordPress site (`wp-content/plugins/plugin-check/prompts/`), read its original prompts, apply the stricter rule wherever they differ from this skill's rule files, and note each difference.
  - If a `wp-core-reference` skill is available, load it and use the installed core (`wp-includes/`, `wp-includes/script-loader.php`) to confirm which libraries and script handles core ships before flagging WP.org 13.
- **Tools**: the project's PHPCS (with its WordPress ruleset if configured) and PHPStan commands. If WP-CLI and the Plugin Check plugin are both installed, run `wp plugin check <slug>` too; never install either. Before using any warning from these tools as evidence, judge it with the rule file `plugin-check/README.md` maps its sniff code to. Warnings the rule file shows to be false positives are not evidence and never become TODO rows.
- **Map**:
  - Identity: slug, main file and its location, every header field, readme header fields, text domains used in gettext calls, versions (header, Stable tag, constants), and the names registered (post types, taxonomies, query vars, options, transients, custom hooks, block names, function, class and namespace prefixes).
  - Package: bundled libraries with license and version, minified or built files with their source and build tool (or the readme link to them), fonts and images with their license, files the deploy config excludes.
  - Outbound: every remote call and remote asset, with its URL or host, trigger (activation, `admin_init`, cron, page view, user action) and the consent or setting that guards it.
  - Admin surfaces: notices, dashboard widgets, menus, upsell and ad output, with the screens they show on and how they are dismissed.
  - Front-end output: credits, links and branding printed on visitor pages, and the setting that enables them.
  - Gates: license keys, plan or "pro" checks, trials, quotas, and update checkers or installers.
  - Platform: PHP extensions used and their fallbacks, `.htaccess`, `php.ini` or multisite assumptions.
- **Extra scope**: none.
- **Cross-slice**: the same consent check applied on every outbound path; versions and text domains consistent across files; the same feature gated in one file and open in another; a library bundled twice; names that collide with reserved terms or core globals across files.
- **Lenses**:
  1. Outbound and remote code (WP.org 7 and 8): every remote call, remote asset, iframe, update checker and installer, traced from its trigger to its consent check.
  2. Gates, upsells and output (WP.org 5, 6, 9, 10, 11; WP.com: WordPress.com experience): feature gates, trials and quotas, admin notices and ads, front-end credits, and compliance claims in strings, readme and assets.
  3. Package and identity (WP.org 1, 2, 4, 12, 13, 15, 17; WP.com: Coding standards and best practices, Main file naming, Text domains, Readme, Plugin author name, Version management): licenses of every shipped file, readable source, bundled core libraries, readme and header fields, versions, slug, names and text domain.
  4. WordPress.com platform (only when `TARGET` includes `com`; WP.com: Custom database tables and data storage, Prevent data leaks, Logging data, Reserved terms, Avoid global variable conflicts, Optimize custom Query, PHP extension requirements): PHP extensions and fallbacks, server assumptions, data storage, reserved terms, globals and logging.
- **Extra report**: the `TARGET` used; whether the plugin depends on WooCommerce and whether it is a Block Plugin; each guideline the Criteria list as not checkable from code, marked "not checked"; the source pages and their dates from the Criteria; the Plugin Check and PHPCS warnings dismissed as false positives, each with its sniff code, `path:line`, rule file and reason; any difference found between Plugin Check's original prompts and this skill's rule files; and, always, "For full security coverage, run `/exact-skills:exact-review-php` on this folder."
- **Example**:

  > `includes/class-telemetry.php:42` (`Telemetry::ping`)<br><br>**WP.org 7: Tracking and consent**: plugins "may not contact external servers without explicit and authorized consent". Hooked on `admin_init`, this posts `home_url()` and the admin email to `https://api.example.com/ping` on every admin page load, and no setting or opt-in guards it, so a fresh install sends data the first time an admin opens the dashboard. Add a "Share usage data" setting that defaults to off, return early from `ping()` unless it is on, and describe the data sent in the readme. Keep the payload and endpoint unchanged. Verify: on a fresh install, load wp-admin and confirm no request to api.example.com (Query Monitor's HTTP panel); enable the setting and confirm one request.
