# Review criteria

Shared by the main agent and every slice sub-agent.

## Sources

The rules below come from these pages. The two guideline pages were read in full; for API reference pages (`wp_enqueue_script`, `register_taxonomy`, `register_post_type`) and the GNU list, only the sections cited were used. When a page and this file disagree, the page wins; note it in the final reply.

- **WP.org**: [Detailed Plugin Guidelines](https://developer.wordpress.org/plugins/wordpress-org/detailed-plugin-guidelines/) (last updated March 11, 2026), and the pages it links: [Block Specific Plugin Guidelines](https://developer.wordpress.org/plugins/wordpress-org/block-specific-plugin-guidelines/), [How your readme.txt works](https://developer.wordpress.org/plugins/wordpress-org/how-your-readme-txt-works/), [the readme.txt standard](https://wordpress.org/plugins/readme.txt), [How to use Subversion](https://developer.wordpress.org/plugins/wordpress-org/how-to-use-subversion/), [Compliance Disclaimers](https://developer.wordpress.org/plugins/wordpress-org/compliance-disclaimers/), [Plugin Assets](https://developer.wordpress.org/plugins/wordpress-org/plugin-assets/), [default scripts in `wp_enqueue_script`](https://developer.wordpress.org/reference/functions/wp_enqueue_script/#default-scripts-and-js-libraries-included-and-registered-by-wordpress), [GPL-compatible licenses](https://www.gnu.org/licenses/license-list.html#GPLCompatibleLicenses), [WordPress Foundation trademark policy](https://wordpressfoundation.org/trademark-policy/).
- **WP.com**: [Plugin developer guidelines](https://developer.wordpress.com/docs/wordpress-com-marketplace/plugin-developer-guidelines/) (last updated October 1, 2025), which calls itself "guidelines and not restrictions", and the pages it links: [Plugin best practices](https://developer.wordpress.org/plugins/plugin-basics/best-practices/), [How to internationalize your plugin](https://developer.wordpress.org/plugins/internationalization/how-to-internationalize-your-plugin/), [`plugin_action_links_{$plugin_file}`](https://developer.wordpress.org/reference/hooks/plugin_action_links_plugin_file/), [Custom hooks](https://developer.wordpress.org/plugins/hooks/custom-hooks/), [Data validation](https://developer.wordpress.org/apis/security/data-validation/), [Global variables](https://developer.wordpress.org/apis/global-variables/), reserved terms in [`register_taxonomy`](https://developer.wordpress.org/reference/functions/register_taxonomy/#reserved-terms) and [`register_post_type`](https://developer.wordpress.org/reference/functions/register_post_type/#reserved-post-types), [Checking user capabilities](https://developer.wordpress.org/plugins/security/checking-user-capabilities/), [WP.com server environment](https://wordpress.com/support/php-environment/), [Version numbering](https://make.wordpress.org/core/handbook/about/how-the-release-cycle-works/version-numbering/). Several WP.com links point to codex.wordpress.org or woo.com pages that have moved; the pages above are where they live now.

- **Plugin Check**: the rules the WordPress.org Plugins Team's Plugin Check 2.2.0 uses to judge its own warnings and plugin names, rewritten in `plugin-check/` next to this file (see its `README.md` for the sniff-code table).

Apply WP.org categories when the target is `org` or `both`, WP.com categories when it is `com` or `both`.

## Review areas

Each `###` heading below is a Category, in Category order. WP.org categories carry the guideline's number; WP.com categories carry the WP.com page's own section title, since that page doesn't number them. Each has a default severity; adjust it with the Severity in context rule. Check what applies; skip what doesn't. Paths such as `plugin-check/naming.md` are relative to this file's folder; read a rule file before judging the cases it covers.

### WP.org Developer Expectations: Security

Default: Critical when a visitor or lower-privileged user can exploit it; High otherwise. The WP.org page: "All code in the directory should be made as secure as possible", and "Should a plugin be found to have security issues, it will be closed until the situation is resolved."

Check at guideline level, using the Plugin Check rule files for what is and isn't an issue:

- Output printed without late escaping (`plugin-check/late-escaping.md`).
- Request data processed without a nonce, or with a nonce but no capability check (`plugin-check/nonce-verification.md`).
- Input stored or used without sanitizing (`plugin-check/sanitization.md`).
- SQL with variable data that isn't prepared (`plugin-check/direct-db-queries.md`).
- `register_setting()` without a sanitize callback (`plugin-check/setting-sanitization.md`).

### WP.org 1: GPL license

Default: High. "All code, data, and images ... must comply with the GPL or a GPL-Compatible license", including bundled third-party libraries, images and fonts.

- The main plugin header and `readme.txt` declare a `License` (GPLv2 or later recommended) and the two agree.
- Every bundled library (`vendor/`, copied JS or CSS, fonts, images, icon sets) has a license, and that license is in the GPL-compatible section of the GNU license list. Read the library's own license file, `composer.lock` or `package-lock.json` license fields, or the file header; don't guess from the name.
- GNU verdicts that come up often in plugins (GNU license list, snapshot of November 17, 2026):
  - Compatible with GPLv2 and GPLv3: Expat (often called MIT), X11, ISC, modified (3-clause) BSD, FreeBSD (2-clause), zero-clause BSD, LGPL 2.1, MPL 2.0, Artistic 2.0, zlib, Boost, the Unlicense, WTFPL 2, public domain. For non-software works: CC0 and CC BY 4.0.
  - Compatible with GPLv3 only, so not with a "GPLv2 only" plugin: Apache 2.0, LGPL 3, AGPL 3, Freetype, Educational Community License 2.0, XFree86 1.1. A "GPLv2 or later" plugin may combine them under GPLv3.
  - Incompatible: the original 4-clause BSD, Apache 1.0 and 1.1, MPL 1.1, EPL 1.0 and 2.0, CDDL, EUPL 1.1 and 1.2, the PHP License 3.01, the OpenSSL license, Ms-PL, Ms-RL, ODbL for data.
  - Nonfree: no license at all, the JSON License ("Good, not Evil"), Commons Clause, Artistic 1.0, the Hippocratic License, the Anti-996 License.
  - Fonts: the SIL Open Font License 1.1 is listed as a free font license, not in the GPL-compatible list. Record bundled OFL fonts with Medium confidence and quote this, so the team can decide.
- No file says "all rights reserved", "proprietary" or forbids redistribution.

### WP.org 2: Developer responsibility

Default: High. Developers must confirm "the licensing of all included files" and "comply to the terms of use for all third party services and APIs". "If there is no way to validate the licensing of a library or the terms of an API, then they cannot be used."

- A bundled file with no license or source anywhere (no header, no manifest entry, no upstream link).
- A third-party API called with no terms of use you can name. Report what is missing; don't judge the terms.
- Code that restores something WP.org asked to remove is out of reach; skip it.

### WP.org 4: Human-readable code

Default: High; Critical when the hidden code does something the readable code doesn't (remote calls, `eval`, file writes).

- Obfuscated code, as defined in `plugin-check/code-obfuscation.md`, plus ionCube or other encoders, `gzinflate` chains and unclear names such as `$z12sdf813d`.
- Minified or bundled JS and CSS are fine only when the readable source and build tools ship in the plugin or the readme links the public development repository. Flag shipped `*.min.js`, `build/` or `dist/` output with neither.
- Recommended, Low: document how to run the build tools.

### WP.org 5: Trialware

Default: High. "Plugins may not contain functionality that is restricted or locked, only to be made available by payment or upgrade."

- Code that exists in the plugin but runs only with a license key, a paid plan or an "is_pro" flag.
- Features disabled after a trial period, date or usage quota.
- A plugin that only gives sandbox or test access to an API.
- Fine: paid features delivered by an external service (see WP.org 6), and premium code kept in a separate add-on plugin not hosted on WP.org. Developer tools are reviewed case by case; say so instead of flagging.

### WP.org 6: Software as a Service

Default: High. Services are allowed, even paid ones, when the service provides "functionality of substance" and is documented in the readme, preferably with a link to its Terms of Use.

- A remote service whose only job is to validate license keys while all functionality is local.
- Code moved to a remote server only so the plugin appears to need the service.
- A storefront: a plugin that only fronts products sold elsewhere.
- A service the plugin depends on that the readme doesn't name or explain. Default Medium.

### WP.org 7: Tracking and consent

Default: Critical. Plugins "may not contact external servers without explicit and authorized consent", usually an opt-in (registration with a service or a settings checkbox). Data use belongs in the readme, ideally with a privacy policy.

- Trace every outbound call (`wp_remote_*`, `wp_safe_remote_*`, `download_url`, cURL, `file_get_contents` or `fopen` on URLs, `fsockopen`, sockets) and every remote URL printed into a page (scripts, styles, images, iframes, pixels). For each, name the trigger (activation, `admin_init`, cron, page view) and the consent check that guards it. Flag any that run before opt-in.
- Data collected without the user confirming it: site URL, admin email, user lists, plugin lists, server info.
- Users misled into submitting information "as a requirement for use of the plugin".
- Remote assets "unrelated to a service".
- Remote data such as blocklists that the readme doesn't document.
- Third-party ad code that tracks usage or views.
- Fine: a service plugin (the guideline names Twitter, an Amazon CDN plugin, Akismet) where installing, activating, registering and configuring the plugin is the consent. Only calls to that service are covered; other calls still need consent.

### WP.org 8: Executable code from third parties

Default: Critical for updates or installs from elsewhere; High for the rest. Loading code from documented services is allowed, but "all communication must be made as securely as possible".

- Updates or installs of plugins, themes or add-ons from any server other than WP.org: a custom updater as defined in `plugin-check/plugin-updater.md`, a `plugins_api` filter, or `Plugin_Upgrader` or `WP_Upgrader` fed a non-WP.org URL.
- Installing a premium version of the same plugin.
- JS or CSS from a third-party CDN. "All non-service related JavaScript and CSS must be included locally"; fonts are the one exception.
- Regularly updated lists fetched from a third party whose terms don't allow it.
- Admin pages connected through iframes instead of APIs.
- Plain `http://` for code or data where HTTPS is available.
- Fine: a management service that pushes software from its own domain, "not within the WordPress dashboard".

### WP.org 9: Illegal or dishonest behaviour

Default: High; Critical for using the user's server or resources without permission (crypto-mining, botnets).

- Claims that the plugin creates, provides, automates or guarantees legal compliance (GDPR, ADA, EU Cookie Law, VAT, HIPAA, PCI and so on). Check the plugin name, readme, descriptions, admin strings and any banners or screenshots in the folder. "Helps you comply" wording is fine; "100% compliant" is not. A service may say it handles compliance when the readme makes clear it is the service, with dated proof linked.
- Wording that implies users must pay to unlock features already in the plugin.
- Descriptions that claim the plugin is better than others, the best, the first or the only solution ("#1 plugin for…", "the only plugin for…"). Default Medium. Rules: the description section of `plugin-check/naming.md`.
- Offers to drive traffic to sites that use the plugin.
- Prompts that pressure or reward users for reviews.
- Conduct items (fake reviews, harassment, code-of-conduct violations, disguised identities) can't be seen in code; leave them out.

### WP.org 10: Credits and links on the public site

Default: High.

- "Powered by" text, credits or links printed on the front end (`wp_footer`, `the_content`, widgets, shortcode, block output, emails sent to visitors) unless an opt-in setting turns them on and that setting defaults to off.
- An opt-in buried in the terms of use or documentation instead of a clear choice in the settings.
- A plugin that refuses to work, or degrades, unless the credit is shown.
- Fine: a service brands its own output when that branding comes from the service, not the plugin code.

### WP.org 11: Admin dashboard hijacking

Default: Medium.

- Upgrade prompts, notices and alerts that aren't limited in scope: shown on every admin screen when they belong on the plugin's screens.
- Site-wide notices or dashboard widgets that can't be dismissed and don't dismiss themselves when resolved. Check that dismissal is stored and honoured.
- Error messages that don't say how to fix the problem, or don't go away once it is fixed.
- Advertising in the dashboard: "should be avoided". Referral tracking in those ads also breaks WP.org 7.
- Fine: links to the developer's own site or social networks, and images bundled in the plugin.

### WP.org 12: Readme spam

Default: Medium. "Readmes are to be written for people, not bots."

- More than 5 tags; tags naming competitors (related products are fine, so a WooCommerce extension may use `woocommerce`); tags unique to the plugin, which aren't shown.
- Keyword stuffing or black-hat SEO: the same term repeated in the plugin name, the readme or translation files.
- Affiliate links that aren't disclosed, or that go through a redirect or cloaked URL.
- Links to products the plugin doesn't need, beyond a moderate number.

### WP.org 13: WordPress default libraries

Default: High. Plugins "may not include those libraries in their own code" and "must use the versions of those libraries packaged with WordPress". The guideline names jQuery, Atom Lib, SimplePie, PHPMailer and PHPass.

- A bundled copy of a library core ships, in PHP (`vendor/` included) or JS. Confirm against the installed core: the scripts registered in `wp-includes/script-loader.php` and the libraries in `wp-includes/`. The table on the `wp_enqueue_script` page is partial and out of date.
- A script enqueued from the plugin, or from a CDN, when core has the same handle (`jquery`, `jquery-ui-*`, `underscore`, `backbone`, `wp-codemirror`, `imagesloaded`, `masonry` and so on).
- `wp_deregister_script( 'jquery' )` followed by registering another copy.

### WP.org 15: Version numbers and readme

Default: Medium. "Users are only alerted to updates when the plugin version is increased. The trunk readme.txt must always reflect the current version of the plugin."

- The `Version` header in the main plugin file and `Stable tag` in `readme.txt` disagree, or a version constant the plugin defines disagrees with them. The directory takes the version from the main plugin file, not the readme.
- `Stable tag: trunk` (prohibited for new plugins), or a Stable tag that isn't only numbers and periods.
- Readme header problems: `Requires at least` and `Requires PHP` missing from the main plugin file header (since WordPress 5.8 they are read from the plugin file, not the readme); `Tested up to` or `Requires PHP` with text such as "WP 6.5" or "PHP 7.4"; `Contributors` that aren't WP.org usernames; a short description over 150 characters or with markup; an upgrade notice over 300 characters; a readme over 10 KB (may cause errors; keep only the current changelog and move the rest to `changelog.txt`).
- The main plugin file in a subfolder of the shipped package, or `.zip` files in the package ("you also should never upload zip files").
- Screenshots: one numbered readme line per `screenshot-N` file, lowercase names. Screenshots, banners and icons belong in the SVN `assets/` directory, not inside the plugin; a deploy folder such as `.wordpress-org/` should use the exact names and sizes from the Plugin Assets page (`banner-772x250`, `banner-1544x500`, `icon-128x128`, `icon-256x256`, `icon.svg` with a PNG fallback, limits of 4 MB, 1 MB and 10 MB).
- The `Plugin URI` header shared between a free and a pro plugin.

### WP.org 17: Trademarks and project names

Default: High. The use of a trademark or another project as "the sole or initial term of a plugin slug is prohibited" unless the author owns it; "Dancing Sloths for Superbox" is fine, "Superbox Dancing Sloths" is not.

- The display name (plugin header and readme), slug and owner: apply `plugin-check/naming.md`, including its severities and its suggested-name fix.
- A block namespace that begins with another product's trademark.
- "WordPress" in a domain the plugin uses for itself. WordPress Foundation word marks: WordPress, WordCamp, BuddyPress, Openverse, WP-CLI.
- Prefer original branding; no finding for that alone.

### WP.org Block Directory

Apply only when the plugin exists solely to distribute one block (the Block Specific Plugin Guidelines call this a Block Plugin). Default: Medium, because a plugin that fails these can still go to the main directory.

- It has UI outside the editor (options pages, admin menus) or only restyles existing blocks.
- It bundles several independent blocks; parent and child blocks are fine.
- Plugin and block titles don't describe the block, or the namespace is reserved (`core`, `wordpress`) or isn't the author or slug.
- `block.json` is missing, or lacks `name`, `title`, one of `script` or `editorScript`, or one of `style` or `editorStyle`.
- It needs another plugin or theme, an account, a login, an activation key or payment; it calls a remote API for work it could do locally.
- It has more server-side PHP than registration needs where the REST API would do.
- It shows alerts, dashboard notices or promotions unrelated to the block.

### WP.com: Coding standards and best practices

Default: Low. A plugin should "adhere to all WordPress plugin coding standards, as well as best practice guidelines for harmonious existence within WordPress and alongside other WordPress plugins". The best-practices page it links asks for:

- Globally accessible names not prefixed: functions and classes, interfaces and traits (unless namespaced), namespaces, global variables, options, transients, custom hook names. The prefix should be unique, at least 4 letters (5 recommended), not a common English word, and not `__`, `_`, `wp_` or `WordPress`. A WooCommerce extension also avoids `woo` and `WooCommerce` prefixes.
- `function_exists()` or `class_exists()` wrapped around the plugin's own functions and classes; that's for shared libraries only.
- Files other than the main plugin file and `uninstall.php` sitting at the plugin root when they could live in subfolders.

### WP.com: WordPress.com experience

Default: Medium. A plugin should "not subvert the WordPress.com experience", for example "inserting spam links or up-selling services outside the WordPress.com ecosystem". Users should see no "plugin advertising" in WP-Admin or the store.

- Upsell banners, pro-version ads, cross-promotion of other products or affiliate links in WP-Admin.
- Spam links in output.

### WP.com: Main file naming

Default: Low. A plugin in folder `plugin-name` has its main file named `plugin-name.php`.

### WP.com: Text domains

Default: Medium (translations break). The text domain matches the plugin folder name, lowercase, with dashes, no underscores and no spaces.

- A text domain in any gettext call that differs from the slug, or more than one text domain.
- The text domain passed as a variable or constant instead of a literal string.
- A gettext call (`__`, `_e`, `_x`, `_n`, `esc_html__` and so on) with no text domain.
- `Text Domain` header that differs from the slug, or a `Domain Path` without its leading slash.

### WP.com: Localization

Default: Low.

- User-facing strings that aren't in English, or that aren't wrapped in gettext.
- Variables interpolated inside translated strings; placeholders without a `translators:` comment; several placeholders without argument numbers (`%1$s`); plurals without `_n()`; sentences built by concatenating translated fragments; empty strings or `\r` in translated strings.
- A plugin aimed at a specific market without translation files for that market.

### WP.com: Follow WordPress PHP guidelines

No rows. The WordPress PHP coding standards (quotes, indentation, brace style, no shorthand PHP tags, Yoda conditions, naming) are PHPCS's job; its output goes in the Tools summary, grouped by rule.

### WP.com: Add Settings action link

Default: Low. A plugin with a settings screen adds a Settings link with `plugin_action_links_{$plugin_file}`, where `$plugin_file` is `plugin_basename()` of the main file.

### WP.com: Custom database tables and data storage

Default: Low; Medium when the choice costs something real.

- A custom table where a post type, taxonomy or option would do.
- Expiring data stored as a permanent option instead of a transient; data that is always needed stored somewhere else instead of an option; entities stored in an option array instead of a post type; categorization stored without a taxonomy.

### WP.com: Prevent data leaks

The WP.com guideline asks for `if ( ! defined( 'ABSPATH' ) ) { exit; }` after the opening tag of every PHP file. Default Medium for a file without it that runs code outside class or function definitions (the case the best-practices page it links names); Low for other PHP files.

### WP.com: Readme

Default: Medium. The plugin has a standard `readme.txt`. Its header fields are checked under WP.org 15 when that target applies; under `com` alone, check them here.

### WP.com: Plugin author name

Default: Low. Products on WordPress.com include `Author` and `Developer` headers naming the person or company, and a `Developer URI`. The WP.com sample header also has `Plugin URI`, `Description`, `Version`, `Author URI`, `Text Domain`, `Domain Path`, `License` and `License URI`.

### WP.com: Make it extensible

Default: Low.

- Settings forms or front-end output that other plugins can't change without editing the code, where an action or filter is the obvious fit. Record only with a concrete case.
- A plugin with blocks suited to layouts but no block patterns. Record only when the case is clear.

### WP.com: Remove unused code

Default: Low. Commented-out code ("there's no reason to leave commented-out code").

### WP.com: Comment effectively

Default: Low. Functions without a docblock saying what they do.

### WP.com: Avoid god objects

Default: Medium when the class or function is large enough to hide bugs; Low otherwise. Split by job, never by line count.

### WP.com: Test your code with WP_DEBUG

Default: Low; higher when the notice breaks output or headers. Code that raises PHP notices or warnings under `WP_DEBUG`: unset variables or array keys, deprecated calls. Use PHPStan and PHP lint output as evidence.

### WP.com: Separate business logic and presentation logic

Default: Low. Classes or functions that mix data handling with HTML output, such as settings saving and rendering in one method.

### WP.com: Use transients to store offsite information

Default: Medium on hot paths, Low otherwise. Data fetched from an API on each request with no transient.

### WP.com: Logging data

Default: Low; Medium when logs hold personal data. Logging must be opt-in, default off. When the plugin depends on WooCommerce, logs go through WooCommerce's logger (`WC_Logger`) so users see them on the WooCommerce status page; the WP.com page also shows a settings snippet that links to those logs. Don't apply the `WC_Logger` rule to plugins without WooCommerce.

### WP.com: Data validation

Default: per the security rules of `exact-review-php`; check at guideline level only. "Validate user input as early as possible."

- Input used before it is validated.
- Loose comparisons (`==`, `in_array()` without `true`) against an allow list.
- Input stored or used without sanitizing, and `register_setting()` without a sanitize callback: `plugin-check/sanitization.md` and `plugin-check/setting-sanitization.md`.

### WP.com: Reserved terms

Default: Medium (causes 404s with no explanation).

- Post types, taxonomies, query vars or request field names that use a reserved term: attachment, attachment_id, author, author_name, calendar, cat, category, category__and, category__in, category__not_in, category_name, comments_per_page, comments_popup, custom, customize_messenger_channel, customized, cpage, day, debug, embed, error, exact, feed, fields, hour, link_category, m, minute, monthnum, more, name, nav_menu, nonce, nopaging, offset, order, orderby, p, page, page_id, paged, pagename, pb, perm, post, post__in, post__not_in, post_format, post_mime_type, post_status, post_tag, post_type, posts, posts_per_archive_page, posts_per_page, preview, robots, s, search, second, sentence, showposts, static, status, subpost, subpost_id, tag, tag__and, tag__in, tag__not_in, tag_id, tag_slug__and, tag_slug__in, taxonomy, tb, term, terms, theme, title, type, types, w, withcomments, withoutcomments, year.
- Post types named after core's: post, page, attachment, revision, nav_menu_item, custom_css, customize_changeset, oembed_cache, user_request, wp_block, wp_global_styles, wp_navigation, wp_template, wp_template_part; or action, author, order, theme.

### WP.com: Avoid global variable conflicts

Default: Medium. Code that overwrites a core global (`$post`, `$posts`, `$wp_query`, `$wpdb`, `$wp`, `$wp_rewrite`, `$wp_roles`, `$pagenow`, `$wp_version` and the rest on the Global variables page) or reads globals where an API function exists.

### WP.com: Optimize custom Query

Default: per `exact-review-php`. Direct SQL or custom queries where `WP_Query` and the core query functions would do. Check at guideline level and refer to `exact-review-php` for query performance.

### WP.com: PHP extension requirements

Default: High when the plugin fatals or loses a feature on WordPress.com; Low for a missing fallback that is never reached.

- Functions or classes from extensions WP.com doesn't load. Loaded: apcu, bcmath, calendar, Core, ctype, curl, date, dom, exif, fileinfo, filter, gd, gmagick, gmp, hash, iconv, imap, intl, json, libxml, mbstring, mcrypt, memcache, mysqli, mysqlnd, openssl, pcntl, pcre, PDO, pdo_mysql, pdo_sqlite, Phar, posix, Reflection, session, shmop, SimpleXML, soap, sockets, sodium, SPL, sqlite3, standard, sysvsem, sysvshm, timezonedb, tokenizer, xml, xmlreader, xmlwriter, xsl, Zend OPcache, zip, zlib. Not installed: ImageMagick (`imagick`), ionCube, SUHOSIN.
- An optional extension used without an `extension_loaded()` or `class_exists()` check and a fallback (for example, GD when ImageMagick is missing).
- Reliance on `.htaccess` (WP.com runs nginx), on `php.ini` changes, on multisite (not supported), or on more than 512 MB of memory.
- PHP version: WP.com runs 8.3, 8.4 and 8.5; code that fails on those versions.

### WP.com: Implement nonces

Default: per the security rules of `exact-review-php`; check at guideline level only. If a plugin lets users submit data, it "must verify that the user is who they say they are and that they have the necessary capability".

- A handler that changes data with no nonce check, or with a nonce but no `current_user_can()` check. A nonce is never a permission check. Exceptions: `plugin-check/nonce-verification.md`.
- `is_admin()` used as if it were a permission check.

### WP.com: Minimize security risks

Default: per the security rules of `exact-review-php`. The page asks developers to "implement maximum security while minimizing security risks" and points to the OWASP Top Ten. Check at guideline level only:

- Output printed without late escaping: `plugin-check/late-escaping.md`.
- SQL with variable data that isn't prepared: `plugin-check/direct-db-queries.md`.

### WP.com: Version management

Default: Low. The plugin version is a dotted number (WP.com points to WordPress's version numbering), and the versions in the plugin agree (see WP.org 15).

## Not checkable from code

Don't record these as tasks. List them in the final reply as not checked:

- WP.org 3 (the stable version is the one in the directory), 14 (commit frequency), 16 (complete at submission), 18 (the team's rights).
- WP.org 2 restored code, and the conduct parts of WP.org 9.
- Developer contact details on WP.org (no auto-replies or support-system addresses).
- Whether the name is too similar to an existing plugin in the directory. That needs a WordPress.org search, which this skill doesn't run.
- WP.org 12 when no `readme.txt` or translation files ship.

## What not to flag

- Anything a guideline explicitly allows: fonts from a CDN; calls to the service a SaaS plugin exists for, once configured; paid features that live in the service; premium code in a separate add-on; WP.org upsells within WP.org 11 (when the target is `org` only); links to the developer's own site in the admin; readme updates that only bump `Tested up to`.
- `global $post; ... setup_postdata(); ... wp_reset_postdata();` and other patterns core documents for globals.
- Style or formatting that PHPCS governs.
- Security, performance and correctness issues beyond the guideline-level checks in WP.org Developer Expectations: Security and WP.com: Implement nonces, Data validation, Minimize security risks and Optimize custom Query; those belong to `exact-review-php` and `exact-review-js`.
- Files that the deploy config (`.distignore`, `export-ignore` in `.gitattributes`, the build or zip script) keeps out of the shipped package, except as evidence for WP.org 4 (source availability).
- Anything in the excluded paths.
- Code marked with a `shortcut:` or `ponytail:` comment that names its limit; that is a decision, unless the real load already crosses the limit.

## Finding contract

Every finding has:

- **Location**: `path:line` plus the class and method or function name, or the file and field for headers, readme lines and manifests.
- **Guideline**: the category, and a short quote of the guideline's own words that the code breaks, at the start of the Instructions.
- **Evidence**: what the code or file does, and a concrete scenario where it breaks the guideline (inputs or state, then the outcome, such as "on activation it posts the admin email to api.example.com with no opt-in"). "Could" or "might" without a scenario is not a finding.
- **Verification**: you read the relevant code, including callers, the hooks that reach it, and the deploy config that decides whether the file ships.
- **Severity**: start from the category's default.
  - Critical: users are harmed now, or the plugin would be closed (tracking without consent, code installed from a third party, resource abuse, obfuscated code that hides behaviour).
  - High: breaks a WP.org "must" or "may not" and blocks approval, or makes the plugin fail on WordPress.com.
  - Medium: breaks a WP.org "should" or a readme rule, or a WP.com guideline with a real cost to users.
  - Low: a WP.com best practice or a recommendation.
- **Confidence**: High (the breach follows directly from the code) or Medium (the code is verified, but whether it breaks the guideline depends on something you can't confirm, such as a service's terms or whether a setting is opt-in elsewhere). Drop anything lower.
- **Fix**: the smallest change that complies, chosen with the ponytail ladder: does it need to exist → already in the codebase → PHP or WordPress core → installed dependency → one line → minimum code. Keep behaviour and public APIs, including fired hooks, unless the behaviour itself is the breach.
- **Fix check**: the fix works on the minimum WordPress and PHP versions, and doesn't break another guideline (for example, moving a CDN script local must keep its license).
- **Severity in context**: if the guideline itself carves out the case (a service plugin, a developer tool reviewed case by case), lower the severity or drop the finding, and say why.
- **More than one category**: when one finding fits several categories (in one target or across both), record it once, under the category with the higher severity; on a tie, under the more specific one. Name the other guidelines it breaks in the Instructions.
- **Category**: one of the `###` headings above, exactly as written.
