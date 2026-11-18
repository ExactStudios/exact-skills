# Any other warning

Source: Plugin Check 2.2.0, `prompts/ai-review-generic.md`, rewritten.

For a warning with no specific rule file, decide whether it is a real problem or a false positive.

- Read the whole function and file, not just the flagged line.
- Check whether code elsewhere in the same function or file already handles it.
- A common, accepted WordPress pattern may be a false positive; name the pattern.
- Consider whether the warning applies where the code runs: admin-only code, WP-CLI, cron.
