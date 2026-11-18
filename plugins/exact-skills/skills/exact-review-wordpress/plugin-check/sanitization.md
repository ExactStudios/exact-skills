# Sanitization

Source: Plugin Check 2.2.0, `prompts/ai-review-sanitization.md`, rewritten.

A sanitization issue is user input stored or used without being sanitized.

## Real issue

- A value from `$_POST`, `$_GET`, `$_REQUEST`, `$_SERVER` or `$_COOKIE`, including any array key of them, is stored or used with no sanitizing function.
- The only "sanitizing" is `isset()`, `empty()` or `wp_unslash()`. None of them sanitize.

## Not an issue

- The value goes through a sanitizing function: `sanitize_text_field()`, `sanitize_email()`, `sanitize_file_name()`, `sanitize_title()`, `sanitize_url()`, `absint()`, `intval()`, `wp_kses()`, `wp_kses_post()` and the like.
- The value is type cast (`(int)`, `(float)`, `(bool)`); that sanitizes for that type.
- The value is passed straight to a function that sanitizes it itself, such as `update_option()` for an option registered with a sanitize callback.
- The value is a password, token, secret, signature or API key (judge by the array key, such as `password`, `pwd`, `pass`, `token`, `secret`, `signature`, `api_key`, and by how it is used). Sanitizing can change such values and break authentication. It is fine when the value is only stored hashed (`wp_hash_password()`) or compared (`wp_check_password()`, `hash_equals()`) and is never printed or put into unprepared SQL.

## Lower risk

- The value is only compared with a literal (`'delete' === $_GET['action']`). Sanitizing is still recommended; record it as Low at most.
