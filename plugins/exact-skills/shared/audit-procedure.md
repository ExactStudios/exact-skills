# Audit procedure

Shared by the exact-review skills. The calling skill gives you a **profile**: Arguments, Criteria, Source, Exclusions, Project files, Tools, Map, Extra scope, Cross-slice and Example, plus Extra prep when it has one. Wherever this procedure names a profile field, use the calling skill's value.

## Prerequisite: ponytail

Before anything else, check that ponytail is available: in Claude Code the `ponytail:ponytail` skill; in Cursor a `ponytail` skill or the ponytail ruleset already in your context. If it is missing, stop and ask the user to install it, then run the skill again:

- Claude Code: `/plugin marketplace add DietrichGebert/ponytail`, then `/plugin install ponytail@ponytail`.
- Cursor: `git clone https://github.com/DietrichGebert/ponytail` and `node ponytail/scripts/cursor-hooks.js install`, or `npx skills add DietrichGebert/ponytail --agent cursor`.

## Inputs

The profile's Arguments hold the user's input. The first word is `FOLDER`, the second is `TODO_FILE`. If Arguments shows the literal text `$ARGUMENTS`, your agent doesn't substitute it; take both values from the user's message.

If either is missing, ask for it before doing anything else; suggest `TODO-<folder-slug>.md` as the default `TODO_FILE`. Resolve relative paths from the repository root. Confirm `FOLDER` exists and contains Source files. These are the only questions you ask. Once the audit starts, don't stop to ask; record any assumption in the affected task's instructions.

## Hard rules

- Analysis only. The only file you may create or modify is `TODO_FILE`. No edits to source, tests, config or lockfiles; no fixers (`--fix`, `phpcbf`, formatters); no installs; no commits; nothing posted anywhere.
- Read any file for context, but record findings only for in-scope files in `FOLDER` and for the profile's Extra scope.
- Every recorded finding meets the Finding contract in the Criteria file. Recording nothing for an area is a valid outcome. Never pad.

## 1. Prepare

1. Run `git status --porcelain` and keep the output as the baseline.
2. Load ponytail at level `full` (Claude Code: the `ponytail:ponytail` skill with args `full`). Its ladder governs every Fix. Where it conflicts with this procedure on scope or output format, this procedure wins.
3. Read the Criteria file. Resolve its absolute path now; sub-agents need it.
4. Read `CLAUDE.md`, `.claude/rules/*` and the lint config. Treat project rules as review criteria.
5. Read the Project files to learn what the profile lists for them.
6. If the profile has Extra prep, do it.
7. List the Source files in `FOLDER`. Exclude the profile's Exclusions, vendored or third-party code (`vendor/`, `packages/`) and generated files. Note the excluded paths for the final summary. Test files are read to judge coverage, not reviewed for style.
8. Run the profile's Tools on `FOLDER` without fixing. Use the output as evidence; group it by rule instead of turning each warning into a row.

## 2. Map

Before judging anything, build a short map in your working notes (not in `TODO_FILE`) covering the profile's Map items, plus:

- Consumers outside `FOLDER` (search the repo). Anything consumed outside `FOLDER` is a public API.
- Load: where each entry point runs. Admin code runs for one user at a time, frontend code on every visitor page view, background code in cron or WP-CLI.

## 3. Review

If the in-scope source is at most 25 files and 4,000 lines, review it yourself against the Criteria file. Otherwise split it by subfolder into slices about that size, keeping tightly coupled files together, and dispatch one sub-agent per slice (Claude Code: `general-purpose`), at most 8 in parallel, with the Sub-agent brief below. If your agent can't run sub-agents, review the slices one at a time yourself. After the slices return, review the profile's Cross-slice concerns yourself.

## 4. Verify and consolidate

1. Re-read the code for every Critical and High finding and every Medium-confidence finding from sub-agents. Drop what doesn't hold. With more than 15 findings to check and sub-agents available, hand the re-check to them in batches, giving each the findings and the Criteria file path.
2. Merge duplicates. Combine findings only when one change fixes them all; never bundle unrelated issues.
3. Sort by Severity, then Category.

## 5. Write `TODO_FILE`

If the file exists, read it first: continue the ID sequence from the highest existing ID, skip findings already recorded, and leave existing rows untouched. If its table has different columns, add a new section headed with today's date below it, holding a table in the layout here. Otherwise create the file with a heading that names `FOLDER` and today's date.

| ID | Severity | Category | File | Confidence | Status | Instructions |
|----|----------|----------|------|------------|--------|--------------|

- **ID**: `TASK-001`, `TASK-002`, and so on.
- **File**: `path:line`, comma-separated when there are several.
- **Status**: always `Pending`.
- **Instructions**: a single line with `|` escaped as `\|`. Say what is wrong, why it matters (the failure scenario), what to change, what must not change, and how to verify it, specifically enough that someone can implement it without auditing again. The profile's Example shows the level of detail.

Bad:

> Improve performance of this class.

## 6. Finish

1. Run `git status --porcelain` and compare it with the baseline. Only `TODO_FILE` may differ. If anything else changed, report it; don't try to repair it.
2. Reply with the `TODO_FILE` path, counts by severity, areas reviewed with no findings, excluded paths, assumptions made and the Tools summary. If the folder has security-sensitive code, suggest running `/claude-security` on `FOLDER` for deeper analysis.
3. Stop. Don't implement anything.

## Sub-agent brief

Fill this in for each slice:

> You are auditing part of a codebase. **Read-only: don't create, edit or delete any file, and don't run fixers or installs.**
> Load ponytail at level `full` (Claude Code: the `ponytail:ponytail` skill with args `full`) and use its ladder for every Fix; this brief wins on scope and output format.
> Read `<absolute Criteria file path>` and apply its Review areas, What not to flag and Finding contract.
> Project rules to apply: <paths to CLAUDE.md and the .claude/rules files>.
> Context map: <the map items for your files, including their load>.
> Your files: <list>. Read any other file for context, but report findings only for your files and for <the profile's Extra scope, as it applies to your files>.
> Return only a markdown table with the columns `Severity | Category | File | Confidence | Instructions`, following the Finding contract, and one line listing the areas with no findings. An empty table is a valid answer.
