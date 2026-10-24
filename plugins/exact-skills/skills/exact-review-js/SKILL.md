---
name: exact-review-js
description: Use when the user asks for a read-only audit of the JS, JSX, TS or TSX code in one folder, with the findings recorded as Pending tasks in a TODO markdown file.
argument-hint: "<folder> <todo-file>"
disable-model-invocation: true
---

# Exact Review JS

Audit one folder of JavaScript/TypeScript and record verified, actionable findings in a TODO file. Analysis only.

Read `../../shared/audit-procedure.md` (relative to this skill's folder) and follow it with this profile.

## Profile

- **Arguments**: `$ARGUMENTS`
- **Criteria**: `review-criteria.md` in this skill's folder
- **Source**: `.js .jsx .mjs .cjs .ts .tsx`
- **Exclusions**: build output (directories from the build config and `.gitignore`), minified files, `.d.ts` files, `node_modules`.
- **Project files**: `package.json`, the build config and the test config, to learn the build output directories, how `@wordpress/*` packages are externalized, the JS test framework, and the lint and type-check scripts.
- **Tools**: the project's JS lint and type-check scripts.
- **Map**:
  - Entry points and how they load: script handles, block registration, mount points.
  - Module graph: shared hooks, components and utilities.
  - State: where it lives (local, context, `@wordpress/data` stores) and what writes to it.
  - Server contracts: every REST/AJAX call, its endpoint, and the PHP handler that serves it.
- **Extra scope**: the PHP handlers of REST/AJAX calls made from in-scope files.
- **Cross-slice**: duplication across modules, architecture, dependency direction and bundle composition.
- **Example**:

  > `UserTable.jsx:42` (`UserTable`) refetches on every filter-field change because the effect depends on the whole `filters` object, so changing three fields sends three requests and an older response can overwrite a newer one. Trigger the request from the filter-submit handler instead. Keep the request parameters and initial-load behavior unchanged. Verify with tests for initial load, submit, reset and API failure.
