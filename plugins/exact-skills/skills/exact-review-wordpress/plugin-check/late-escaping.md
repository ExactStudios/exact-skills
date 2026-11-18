# Late escaping

Source: Plugin Check 2.2.0, `prompts/ai-review-late-escaping.md`, rewritten.

An escaping issue is data printed without being escaped. Read the flagged line and follow the value back to where it is built.

## Real issue

- The value is printed with no escaping function at the point of output.
- The value was escaped earlier into a variable and printed later. That is not late escaping; escape in the output statement itself.
- The only wrapper is a translation function (`__()`, `_e()`, `_x()` and the like) or `printf()`/`sprintf()`. None of them escape.

## Not an issue

- The output is a hard-coded string with no variables.
- The output is the direct return of an escaping function: `esc_html()`, `esc_attr()`, `esc_url()`, `esc_js()`, `esc_textarea()`, `wp_kses()`, `wp_kses_post()`, `wp_kses_data()`, or the translate-and-escape variants (`esc_html__()` and so on).
- The value comes from a core function that escapes its own output, such as `get_avatar()`, `paginate_links()` or `wp_nonce_field()`. Check the context before dismissing: that holds only for the function's own markup.
