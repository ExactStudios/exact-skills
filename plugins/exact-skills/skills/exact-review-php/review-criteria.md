# Review criteria

Shared by the main agent and every slice sub-agent.

## Review areas

Check what applies; skip what doesn't.

**Security**
- Authorization: every admin action, `admin_post_*` handler, AJAX handler (especially `wp_ajax_nopriv_*`), REST route and form handler checks a capability with `current_user_can`. `is_admin()` only says the request is for an admin page; it is not a permission check. A REST `permission_callback` of `__return_true` on a route that reads private data or changes state is a finding.
- CSRF: establish how each handler authenticates first. admin-ajax, admin-post and form handlers verify a nonce on every state change (`check_admin_referer`, `check_ajax_referer`, `wp_verify_nonce`). REST routes with cookie auth get the `wp_rest` nonce checked by core, so a missing nonce in the callback is not a finding there. A nonce is never a permission check: it must come with the capability check above.
- Input: superglobals are `wp_unslash`ed, then sanitized with the function that matches the type (`sanitize_text_field`, `absint`, `sanitize_key`, `esc_url_raw` and so on). `register_setting` and REST args have a sanitize or validate callback.
- Output: escaped late, where it is printed, with the function that matches the context (`esc_html`, `esc_attr`, `esc_url`, `esc_js`, `wp_kses_post`), including templates and translated strings (`esc_html__`).
- SQL: all dynamic SQL goes through `$wpdb->prepare()`, including `LIKE` with `$wpdb->esc_like()` and `IN` lists built from placeholders. Table and column names come from a fixed list.
- Object injection: `unserialize` or `maybe_unserialize` on data a user can control.
- Redirects and requests: `wp_redirect` with user input instead of `wp_safe_redirect`; `wp_remote_*` with a user-supplied URL instead of `wp_safe_remote_*`.
- Files: path traversal in includes and file operations; uploads that skip `wp_handle_upload` or `wp_check_filetype_and_ext`.
- Direct access: PHP files that run code on load without an `ABSPATH` check.
- Secrets or private data returned by REST or AJAX, or localized to the page.

**Correctness and reliability**
- `WP_Error` results never checked with `is_wp_error`; remote responses whose status code is never checked.
- Filters that don't return a value; `accepted_args` lower than the callback uses; hooks added twice.
- Type errors against `strict_types` or declared types; `false` or `null` from `get_option`, `get_post` or `get_post_meta` used as an array or object.
- `switch_to_blog` without `restore_current_blog`; options or tables that ignore multisite where the plugin supports it.
- `time()` or `date()` where site time matters, instead of `current_time`, `wp_date` or `current_datetime`.
- Activation, deactivation and uninstall: schema created with `dbDelta`, cron events unscheduled, stored data removed on uninstall.
- Translations: missing or wrong text domain, variables inside translated strings, `sprintf` placeholders without a `translators:` comment, plurals without `_n`.

**Performance and resources**
- Queries or remote calls inside loops; meta read per item where the meta cache could be primed once.
- `WP_Query` with `posts_per_page => -1` on unbounded data, without `no_found_rows` when not paginating, or loading full posts where `fields => 'ids'` is enough.
- `meta_query`, `tax_query` or `LIKE '%...%'` patterns that can't use an index on large sites.
- Expensive work with no transient or object cache, or a cache that is never invalidated.
- Large autoloaded options; options written on every request.
- Work on `init`, `plugins_loaded` or `admin_init` that only one screen or request type needs; remote calls in those hooks.
- Assets enqueued on every page instead of where they are used.
- Cron scheduled without a `wp_next_scheduled` check; long tasks run inside a page request.
- Memory: whole tables or all posts loaded into arrays; static caches that only grow during cron or WP-CLI runs.

**WordPress integration**
- A core API exists but is reimplemented: Settings API, Options, Transients, HTTP API, Filesystem, `wp_parse_args`, `wp_list_pluck` and so on.
- Deprecated core functions, hooks or arguments for the plugin's minimum WordPress version.
- Direct SQL where a core API exists; schema changes not made with `dbDelta`.
- `admin-ajax.php` used where the project already prefers the REST API.
- Hooks the code fires are a public API: renaming them or changing their arguments breaks add-ons.

**PHP**
- Syntax or functions newer than the minimum PHP version.
- Missing `declare(strict_types=1)` or type declarations where project rules require them.
- Errors suppressed with `@`; a broad `catch` that swallows the error; `extract()`; globals used to hold state.

**Simplicity and maintenance**
- Delete: dead code, unused classes, methods, hook callbacks, options and dependencies. Search the whole repo before claiming anything is unused, including tests, config, hook-name strings and dynamic callbacks such as `[ $this, 'method' ]`.
- Reuse: logic the codebase already has as a helper, trait or service (name the path).
- Native: code or a dependency doing what PHP or WordPress core already does.
- YAGNI: interfaces with one implementation, factories with one product, wrappers that only pass calls through, options nobody sets.
- Merge: near-copies that must change together.
- Split: one class or method doing several unrelated jobs. Split by job, never by line count.
- Hidden dependencies: singletons, static calls and globals that make a class hard to test; circular dependencies.

**Tests**
- Using the project's existing PHP test setup, find untested logic that carries real risk: the findings above, permission checks, sanitization, data writes, edge and failure paths. Say what to mock at the WordPress boundary.
- Tests in the audited folder that are broken, skipped or assert nothing.

## What not to flag

- Style or formatting that PHPCS governs, or a preference between equivalent patterns. Breaks of the project's own rules are not style; flag them.
- Speculative performance ("could be slow at scale") without a concrete cost in this code. Admin code runs for one user at a time; frontend code runs on every visitor page view.
- Anything in the excluded paths.
- Changes that would break project rules.
- Code marked with a `shortcut:` or `ponytail:` comment that names its limit; that is a decision, unless the real load already crosses the limit.

## Finding contract

Every finding has:

- **Location**: `path:line` plus the class and method or function name.
- **Evidence**: what the code does, and a concrete scenario where it fails or costs something (inputs or state, then the outcome). "Could" or "might" without a scenario is not a finding.
- **Verification**: you read the relevant code, including callers and the hooks that reach it.
- **Severity**:
  - Critical: exploitable security issue or data loss.
  - High: functional bug, broken flow, or a severe performance or resource problem users will hit.
  - Medium: reliability gap, or a maintenance problem with real cost.
  - Low: minor improvement, test gap on low-risk code, optional optimization.
- **Confidence**: High (the failure follows directly from the code) or Medium (the code is verified, but the impact depends on runtime conditions you can't confirm). Drop anything lower.
- **Fix**: the smallest change that resolves it, chosen with the ponytail ladder: does it need to exist → already in the codebase → PHP or WordPress core → installed dependency → one line → minimum code. Keep behavior and public APIs, including fired hooks, unless the behavior itself is the bug; if a breaking change is unavoidable, list the affected consumers. State impact in words unless you measured it.
- **Category**: one of Security, Correctness, WordPress, PHP, Performance, Resources, Maintenance, Dependencies, Testing.
