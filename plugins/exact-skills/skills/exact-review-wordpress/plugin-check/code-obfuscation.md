# Code obfuscation

Source: Plugin Check 2.2.0, `prompts/ai-review-code-obfuscation.md`, rewritten.

Obfuscation is code made hard to read on purpose, which WP.org 4 doesn't allow.

## Real issue

- Base64-encoded PHP that is decoded and then executed.
- Strings run through `eval()`. Any use of `eval()` is flagged, whatever the context.
- Encoded variable names.
- `str_rot13()` applied to code that is executed.
- Packed JavaScript, such as Dean Edwards' packer (`eval(function(p,a,c,k,e,...`).

## Not an issue

- Minified JavaScript or CSS. That is not obfuscation; whether its source ships is a separate WP.org 4 check.
- Base64 used for images, fonts or other content that isn't executed.
- Encoded strings used as configuration values, API tokens or data payloads that are never executed.
- `base64_decode()` used to decode data rather than code.
