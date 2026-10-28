---
name: wp-core-reference
description: MUST be used before writing, editing or suggesting any WordPress plugin or theme code. Look up every core function, class, action and filter in the installed stubs first, so nothing is invented or deprecated.
---

# WordPress core reference

Target: latest WordPress, PHP 8.x only.

## Before writing any code that touches WordPress
Before you write the code, find out what core already provides:

1. **Functions and classes:** search `vendor/php-stubs/wordpress-stubs/wordpress-stubs.php`.
   Check the exact name, parameters, types and return value.
2. **Globals:** check `vendor/php-stubs/wordpress-globals/`.
3. **Actions and filters:** search `vendor/wp-hooks/wordpress-core/hooks/`.
   Check the hook name, parameters, and when it fires.
4. **WP-CLI:** check `vendor/php-stubs/wp-cli-stubs/`.
5. Read the docblock for `@since` and `@deprecated`.
   If something is deprecated, use the replacement named in the docblock.

## Rules
- If a core function, class or hook does not appear in the stubs, do NOT invent it.
  Say it wasn't found and propose a real alternative.
- Prefer an existing core function over writing your own helper.
- If the stubs folder is missing, tell me to run:
  `composer require --dev php-stubs/wordpress-stubs php-stubs/wordpress-globals php-stubs/wp-cli-stubs wp-hooks/wordpress-core`
- For block editor (JS), REST API schema, and other things not covered by PHP stubs,
  check developer.wordpress.org instead of guessing.
- Use WordPress APIs rather than raw PHP equivalents (`$wpdb->prepare`, `wp_remote_*`,
  `wp_nonce_*`, Settings API). Sanitize on input, escape on output, check capabilities.
- Prefix all functions, classes, globals and hooks.

Do not run PHPStan or PHPCS unless I ask.
