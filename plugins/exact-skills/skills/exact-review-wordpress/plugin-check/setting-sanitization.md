# Setting sanitization

Source: Plugin Check 2.2.0, `prompts/ai-review-setting-sanitization.md`, rewritten.

A setting sanitization issue is `register_setting()` called without a sanitize callback, so whatever is submitted is saved.

## Real issue

- `register_setting()` has no third argument, or its arguments have no `sanitize_callback`.
- The `sanitize_callback` exists but doesn't validate and sanitize the value before it is saved.

## Not an issue

- The third argument includes a `sanitize_callback` that does the job.
- A `sanitize_option_{$option}` filter sanitizes the option.
- The setting stores only a boolean or integer and the code casts it to that type.

## Lower risk

- The setting has a `type` and `show_in_rest` with a `schema`, so core validates part of it. Explicit sanitizing is still recommended; record it as Low at most.
