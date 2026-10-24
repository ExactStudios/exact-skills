---
name: exact-review-php
description: Use when the user asks for a read-only audit of the PHP code in one WordPress plugin or theme folder, with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file>"
disable-model-invocation: true
---

# Exact Review PHP

Audit one folder of WordPress PHP and record verified, actionable findings in a TODO file. Analysis only.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with this profile.

## Profile

- **Arguments**: `$ARGUMENTS`
- **Criteria**: `review-criteria.md` in this skill's folder
- **Source**: `.php`, including templates and views.
- **Exclusions**: `node_modules`, build output, language files, and `index.php` files that only hold a "Silence is golden" comment.
- **Project files**: `composer.json`, the main plugin file header, the PHPCS, PHPStan and PHPUnit configs, to learn the minimum PHP version ("Requires PHP" or `require.php`), the minimum WordPress version ("Requires at least"), the text domain, the autoloading, the PHP test setup (WordPress test suite, Brain Monkey or WP_Mock), and the lint and static-analysis commands.
- **Extra prep**: if a `wp-core-reference` skill is available, load it and use it to confirm core functions, hooks and deprecations before flagging them.
- **Tools**: the project's PHPCS and PHPStan commands.
- **Map**:
  - Entry points: hooks registered (`add_action`, `add_filter`), REST routes, AJAX actions (note every `wp_ajax_nopriv_*`), `admin_post_*` handlers, admin pages, shortcodes, blocks, cron events, WP-CLI commands, activation, deactivation and uninstall.
  - Class and file graph: autoloading, shared services and helpers.
  - Data: options, post and user meta, custom tables and transients the code reads or writes.
  - Public API: hooks the code fires (`do_action`, `apply_filters`), public functions and classes other plugins or add-ons can call.
- **Extra scope**: none.
- **Cross-slice**: duplication across classes, architecture, dependency direction, hooks registered twice, and the same data written from several places.
- **Example**:

  > `includes/class-stats.php:112` (`Stats::get_by_ad`) builds `WHERE ad_id IN (...)` by imploding `$_GET['ids']` into the SQL, so a request with `ids=1) OR 1=1 --` returns every row. Cast the IDs with `array_map( 'absint', ... )`, build `%d` placeholders with `array_fill`, and pass them through `$wpdb->prepare()`. Keep the result shape unchanged. Verify with a test that sends a non-numeric ID and asserts only valid rows come back.
