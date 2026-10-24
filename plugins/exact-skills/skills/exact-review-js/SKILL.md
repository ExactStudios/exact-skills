---
name: exact-review-js
description: Use when the user asks for a read-only audit of the JS, JSX, TS or TSX code in one folder, with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file>"
disable-model-invocation: true
---

# JS/TS Folder Audit

Audit one folder of JavaScript/TypeScript and record verified, actionable findings in a TODO file. Analysis only.

## Inputs

Arguments: `$ARGUMENTS`. The first word is `FOLDER`, the second is `TODO_FILE`.

If either is missing, ask for it before doing anything else; suggest `TODO-<folder-slug>.md` as the default `TODO_FILE`. Resolve relative paths from the repository root. Confirm `FOLDER` exists and contains JS/TS source. These are the only questions you ask. Once the audit starts, don't stop to ask; record any assumption in the affected task's instructions.

## Hard rules

- Analysis only. The only file you may create or modify is `TODO_FILE`. No edits to source, tests, config or lockfiles; no fixers (`--fix`, formatters); no installs; no commits; nothing posted anywhere.
- Read any file for context, but record findings only for in-scope files in `FOLDER` and for the PHP handlers of REST/AJAX calls made from them.
- Every recorded finding meets the Finding contract in `${CLAUDE_SKILL_DIR}/review-criteria.md`. Recording nothing for an area is a valid outcome. Never pad.

## 1. Prepare

1. Run `git status --porcelain` and keep the output as the baseline.
2. Load the `ponytail:ponytail` skill with args `full`. Its ladder governs every Fix. Where it conflicts with this skill on scope or output format, this skill wins. If ponytail isn't installed, use the ladder written in the Finding contract.
3. Read `${CLAUDE_SKILL_DIR}/review-criteria.md`.
4. Read `CLAUDE.md`, `.claude/rules/*` and the lint config. Treat project rules as review criteria.
5. Read `package.json`, the build config and the test config to learn the build output directories, how `@wordpress/*` packages are externalized, the JS test framework, and the lint and type-check scripts.
6. List the files in `FOLDER` with extensions `.js .jsx .mjs .cjs .ts .tsx`. Exclude build output (directories from the build config and `.gitignore`), minified files, `.d.ts` files, `node_modules`, vendored or third-party code (`packages/`, `vendor/`) and generated files. Note the excluded paths for the final summary. Test files are read to judge coverage, not reviewed for style.
7. If the project has a JS lint or type-check script, run it on `FOLDER` without fixing. Use the output as evidence; group it by rule instead of turning each warning into a row.

## 2. Map

Before judging anything, build a short map in your working notes (not in `TODO_FILE`):

- Entry points and how they load: script handles, block registration, mount points.
- Module graph: shared hooks, components and utilities, plus consumers outside `FOLDER` (search the repo). Anything consumed outside `FOLDER` is a public API.
- State: where it lives (local, context, `@wordpress/data` stores) and what writes to it.
- Server contracts: every REST/AJAX call, its endpoint, and the PHP handler that serves it.
- Load: whether each entry point runs in admin (one user at a time) or on the frontend (every visitor page view).

## 3. Review

If the in-scope source is at most 25 files and 4,000 lines, review it yourself against `review-criteria.md`. Otherwise split it by subfolder into slices about that size, keeping tightly coupled files together, and dispatch one `general-purpose` sub-agent per slice, at most 8 in parallel, with the Sub-agent brief below. After the slices return, review the cross-slice concerns yourself: duplication across modules, architecture, dependency direction and bundle composition.

## 4. Verify and consolidate

1. Re-read the code for every Critical and High finding and every Medium-confidence finding from sub-agents. Drop what doesn't hold. With more than 15 findings to check, hand the re-check to `general-purpose` sub-agents in batches, giving each the findings and the path to `review-criteria.md`.
2. Merge duplicates. Combine findings only when one change fixes them all; never bundle unrelated issues.
3. Sort by Severity, then Category.

## 5. Write `TODO_FILE`

If the file exists, read it first: continue the ID sequence from the highest existing ID, skip findings already recorded, and leave existing rows untouched. If its table has different columns, add a new section headed with today's date below it, holding a table in the layout here. Otherwise create the file with a heading that names `FOLDER` and today's date.

| ID | Severity | Category | File | Confidence | Status | Instructions |
|----|----------|----------|------|------------|--------|--------------|

- **ID**: `TASK-001`, `TASK-002`, and so on.
- **File**: `path:line`, comma-separated when there are several.
- **Status**: always `Pending`.
- **Instructions**: a single line with `|` escaped as `\|`. Say what is wrong, why it matters (the failure scenario), what to change, what must not change, and how to verify it, specifically enough that someone can implement it without auditing again.

Good:

> `UserTable.jsx:42` (`UserTable`) refetches on every filter-field change because the effect depends on the whole `filters` object, so changing three fields sends three requests and an older response can overwrite a newer one. Trigger the request from the filter-submit handler instead. Keep the request parameters and initial-load behavior unchanged. Verify with tests for initial load, submit, reset and API failure.

Bad:

> Improve UserTable performance.

## 6. Finish

1. Run `git status --porcelain` and compare it with the baseline. Only `TODO_FILE` may differ. If anything else changed, report it; don't try to repair it.
2. Reply with the `TODO_FILE` path, counts by severity, areas reviewed with no findings, excluded paths, assumptions made and the lint/type-check summary. If the folder has security-sensitive code, suggest running `/claude-security` on `FOLDER` for deeper analysis.
3. Stop. Don't implement anything.

## Sub-agent brief

Fill this in for each slice:

> You are auditing part of a JS/TS codebase. **Read-only: don't create, edit or delete any file, and don't run fixers or installs.**
> Load the `ponytail:ponytail` skill with args `full` and use its ladder for every Fix; this brief wins on scope and output format.
> Read `${CLAUDE_SKILL_DIR}/review-criteria.md` and apply its Review areas, What not to flag and Finding contract.
> Project rules to apply: <paths to CLAUDE.md and the .claude/rules files>.
> Context map: <entry points, shared modules, state, REST/AJAX endpoints with their PHP handler paths, admin or frontend load>.
> Your files: <list>. Read any other file for context, but report findings only for your files and for the PHP handlers of REST/AJAX calls made from your files.
> Return only a markdown table with the columns `Severity | Category | File | Confidence | Instructions`, following the Finding contract, and one line listing the areas with no findings. An empty table is a valid answer.
