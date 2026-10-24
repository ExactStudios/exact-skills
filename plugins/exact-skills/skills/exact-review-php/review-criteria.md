# Review criteria

Shared by the main agent and every slice sub-agent.

## Review areas

Check what applies; skip what doesn't.

**Security**
- Authorization: every operation that needs privileges (admin actions, `admin_post_*`, AJAX and form handlers, REST routes) checks a capability with `current_user_can` or the project's capability helper. Public endpoints (`wp_ajax_nopriv_*`, a REST `permission_callback` of `__return_true`) are fine when their behavior and data are meant to be public; flag them when they expose protected data or let a visitor or lower-privileged user change state they shouldn't. `is_admin()` only says the request is for an admin page; it is not a permission check.
- CSRF: establish how each handler authenticates first. admin-ajax, admin-post and form handlers verify a nonce on every state change (`check_admin_referer`, `check_ajax_referer`, `wp_verify_nonce`). REST routes with cookie auth get the `wp_rest` nonce checked by core, so a missing nonce in the callback is not a finding there. A nonce is never a permission check: it must come with the capability check above.
- Input: superglobals are `wp_unslash`ed, then sanitized with the function that matches the type (`sanitize_text_field`, `absint`, `sanitize_key`, `esc_url_raw` and so on). `register_setting` and REST args have a sanitize or validate callback.
- Output: escaped late, where it is printed, with the function that matches the context (`esc_html`, `esc_attr`, `esc_url`, `esc_js`, `wp_kses_post`), including templates and translated strings (`esc_html__`).
- SQL: all dynamic SQL goes through `$wpdb->prepare()`, including `LIKE` with `$wpdb->esc_like()` and `IN` lists built from placeholders. Table and column names come from a fixed list.
- Object injection: `unserialize` or `maybe_unserialize` on data a user can control.
- Redirects and requests: `wp_redirect` with user input instead of `wp_safe_redirect`; `wp_remote_*` with a user-supplied URL instead of `wp_safe_remote_*`.
- Files: path traversal in includes and file operations; uploads that skip `wp_handle_upload` or `wp_check_filetype_and_ext`.
- Direct access: PHP files that run code on load without an `ABSPATH` check.
- Secrets or private data returned by REST or AJAX, or localized to the page.
- Keys: a key derived from a constant (`AUTH_KEY`, `*_SALT`) that may be undefined (a fatal on PHP 8), still core's sample phrase (a public key), or rotated (every stored ciphertext unreadable). Check the fallback and what happens to stored ciphertext.
- REST args of `type: object` or `array` with no `properties` or `items`: core validates only what is listed, so any JSON reaches storage.

**Correctness and reliability**
- `WP_Error` results never checked with `is_wp_error`; remote responses whose status code is never checked.
- Filters that don't return a value; `accepted_args` lower than the callback uses; hooks added twice.
- Type errors against `strict_types` or declared types; `false` or `null` from `get_option`, `get_post` or `get_post_meta` used as an array or object.
- `switch_to_blog` without `restore_current_blog`; options or tables that ignore multisite where the plugin supports it.
- `time()` or `date()` where site time matters, instead of `current_time`, `wp_date` or `current_datetime`.
- Activation, deactivation and uninstall: schema created with `dbDelta`, cron events unscheduled, stored data removed on uninstall. Without an `uninstall.php`, core includes the main plugin file and boots every integration to uninstall; `register_uninstall_hook()` called on every request reads a non-autoloaded option.
- Multisite: user meta is network-wide, so per-site data or a per-site uninstall that writes user meta leaks across sites.
- Version: check `@since` for every core function and hook used against "Requires at least", and for hooks also where core fires them in that version; a hook can gain a call site later than its `@since`.
- Races: check-then-act on a status transition (read the status, then update and notify). Say whether the runner (Action Scheduler concurrency, parallel WP-CLI) makes it reachable.
- Translations: missing or wrong text domain, variables inside translated strings, `sprintf` placeholders without a `translators:` comment, plurals without `_n`.

**Round trip and contract**
- Round trip: for every write path (REST, form, AJAX, CLI), trace these values from the request through the core write to storage, the read-back and the output or network call: `\`, `&`, `<3`, `'`, `"`, an upper-case email, a string longer than its column, and an array where a string is expected. Check what core does on the way:
  - `wp_insert_post`, `update_metadata`, `wp_insert_comment`, `wp_insert_term` and `wp_insert_user` unslash their input, so callers must `wp_slash`; REST values arrive unslashed.
  - kses on `content_save_pre`, `title_save_pre` and `pre_comment_content` runs for users without `unfiltered_html`, including user 0 in cron and signed-out visitors. `wp_filter_kses` on `pre_user_email`, `pre_comment_author_email`, `pre_term_name` and the user name fields runs for everyone (core's default filters).
  - `sanitize_text_field` encoding `<`; passwords, which core hashes and checks slashed; column length and case-insensitive collation; a stored value later compared strictly with the raw input.
- Contract: for every field a REST route returns and every meta, option or column the code reads, name its writer and its consumers (JS, add-ons). Flag fields that are hard-coded (`'issues' => []`), read but never written, written but never read, and response changes a client reducer would drop.
- Early returns: for every multi-step write, list what is already written, logged, notified or scheduled at each early return. Flag a result ignored before the next step, writes ordered so a failure leaves partial state, and an audit or activity entry only some paths reach.
- Invariants: when one entry point enforces a rule (read-only state, ownership, status lock), check every sibling write route and data method for the same rule.
- Defaults: stored settings or preferences merged with a shallow `array_replace` or `wp_parse_args` over a nested array lose defaults added in later versions; `array_replace_recursive` merges lists by index.
- Errors: a failure reported as a different, misleading error, such as a failed insert answered with "already exists".

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
- Work at route registration: `rest_api_init` runs on every REST request, including other plugins' routes, so `enum` lists built from globs or large lists (`get_available_languages()`, `timezone_identifiers_list()`) cost on every call. Also check constructors of classes loaded on every request, and assets registered on front-end requests.
- Remote calls in a response path whose failure isn't cached, such as `wp_get_available_translations()`.

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
- **Fix check**: the fix works on the minimum WordPress and PHP versions, for every caller, and for every consumer of a changed response (JS reducers, add-ons). A fix that changes a response shape names the client change it needs.
- **Severity in context**: if core or another open path already allows the same thing (for example, core's `profile.php` changes the email without a password), lower the severity and say why. If nothing reaches the path today, call it hardening.
- **Category**: one of Security, Correctness, WordPress, PHP, Performance, Resources, Maintenance, Dependencies, Testing.
