# Nonce verification

Source: Plugin Check 2.2.0, `prompts/ai-review-nonce-verification.md`, rewritten.

A nonce issue is a form submission or AJAX request processed without a nonce check, or `$_POST`, `$_GET` or `$_REQUEST` read before one.

## Real issue

- A form or AJAX handler processes or saves input with no `wp_verify_nonce()`, `check_admin_referer()` or `check_ajax_referer()` before it. AJAX handlers use `check_ajax_referer()` or `wp_verify_nonce()`; form handlers use `check_admin_referer()` or `wp_verify_nonce()`.
- The handler checks `current_user_can()` but no nonce. A capability check alone is not enough for a form submission.
- The nonce is checked after the input is already used.

## Not an issue

- The nonce is verified earlier in the same function, or in the function that calls it.
- The code reads the input only to display something, not to process or save it. This can be acceptable; say why in the summary.
- The code is a REST API callback with a proper `permission_callback`. REST uses its own authentication and needs no nonce in the callback.
