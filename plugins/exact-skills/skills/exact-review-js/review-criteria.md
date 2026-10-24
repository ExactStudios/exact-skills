# Review criteria

Shared by the main agent and every slice sub-agent.

## Review areas

Check what applies; skip what doesn't.

**Security**
- XSS sinks: `dangerouslySetInnerHTML`, `innerHTML`, `outerHTML`, `insertAdjacentHTML`, jQuery `.html()` and `$(html)`, and `href`/`src` built from data (`javascript:` URLs). Trace whether the value can be user- or server-controlled and whether it is escaped.
- For each REST/AJAX call, open the PHP handler and first establish how it authenticates, then check what that model needs. REST with cookie auth: core verifies the `wp_rest` nonce (sent by `apiFetch`), so check the `permission_callback`. admin-ajax and admin-post: check both a capability check and a nonce check. Always check input sanitization. A nonce is never a permission check, and JS checks are never authoritative.
- Localized data that exposes more to the browser than the page needs.
- Third-party or dynamically loaded scripts.

**Correctness and reliability**
- Unhandled promise rejections; request errors swallowed or shown as success.
- Response shapes assumed without guards where the server can return errors or empty data.
- Races: older responses overwriting newer state, double submits, state updates after unmount.
- Missing loading, error or empty states; UI left inconsistent after a failure.
- Assumptions about globals, localized variables, DOM elements or script load order.

**React**
- Hook rules; missing or wrong effect dependencies; stale closures.
- Effects that should be derived state or event handlers.
- `key` usage in lists; switching between controlled and uncontrolled.
- Components with several unrelated responsibilities; prop drilling that an existing store already solves.
- Memoization missing where there is a visible cost, or present where it buys nothing.

**WordPress integration**
- `@wordpress/*` packages externalized through the dependency-extraction mechanism, not bundled.
- `apiFetch` and the REST API used instead of hand-rolled `fetch` or `admin-ajax.php`, matching the rest of the project.
- Every user-facing string goes through `@wordpress/i18n` with the correct text domain, with no concatenation inside translations.
- Existing `@wordpress/components` or house components used instead of hand-rolled equivalents.

**Performance and resources**
- Duplicate or unnecessary requests; requests inside loops.
- Expensive work during render with a real cost; unthrottled scroll, resize or input handlers.
- Listeners, timers, observers and subscriptions without cleanup.
- Heavy imports: whole libraries for one function, admin code in frontend bundles.

**Accessibility**
- Clickable `div`/`span` instead of buttons or links; keyboard access; visible focus.
- Inputs without labels; icon buttons without accessible names.
- Modals that don't trap and return focus; dynamic messages not announced (`speak` or live regions).
- Incorrect or redundant ARIA.

**Simplicity and maintenance**
- Delete: dead code, unused exports, options and dependencies. Search the whole repo, including tests, config and string or dynamic references, before claiming anything is unused.
- Reuse: logic the codebase already has as a helper, hook or component (name the path).
- Native: code or a dependency doing what the platform or an `@wordpress/*` package already does.
- YAGNI: abstractions with one implementation, wrappers that only pass calls through, options nobody sets.
- Merge: near-copies that must change together.
- Split: one component or function doing several unrelated jobs. Split by job, never by line count.
- Circular imports; dependencies that point the wrong way between layers.

**Tests**
- Using the existing JS test framework, find untested logic that carries real risk: the findings above, data transforms, async and error paths. Don't ask for tests of trivial presentational components.
- Tests in the audited folder that are broken, skipped or assert nothing.

## What not to flag

- Style or formatting that lint or Prettier governs, or a preference between equivalent patterns. Breaks of the project's own rules (for example Tailwind v3 class names in JSX) are not style; flag them.
- Speculative performance ("could be slow at scale") without a concrete cost in this code. Admin screens run for one user at a time; frontend scripts run on every visitor page view.
- Anything in the excluded paths.
- Changes that would break project rules.
- Code marked with a `shortcut:` or `ponytail:` comment that names its limit; that is a decision, unless the real load already crosses the limit.

## Finding contract

Every finding has:

- **Location**: `path:line` plus the function or component name.
- **Evidence**: what the code does, and a concrete scenario where it fails or costs something (inputs or state, then the outcome). "Could" or "might" without a scenario is not a finding.
- **Verification**: you read the relevant code, including callers and the server side where relevant.
- **Severity**:
  - Critical: exploitable security issue or data loss.
  - High: functional bug, broken flow, or a severe performance or resource problem users will hit.
  - Medium: accessibility barrier, reliability gap, or a maintenance problem with real cost.
  - Low: minor improvement, test gap on low-risk code, optional optimization.
- **Confidence**: High (the failure follows directly from the code) or Medium (the code is verified, but the impact depends on runtime conditions you can't confirm). Drop anything lower.
- **Fix**: the smallest change that resolves it, chosen with the ponytail ladder: does it need to exist → already in the codebase → platform or `@wordpress/*` → installed dependency → one line → minimum code. Keep behavior and public APIs unless the behavior itself is the bug; if a breaking change is unavoidable, list the affected consumers. State impact in words unless you measured it.
- **Fix check**: the fix works in the project's supported browsers and WordPress version, for every caller and consumer (other screens, add-ons using a public global or filter), and against the server's real response shape. A fix that needs a server change names it.
- **Severity in context**: if another open path already allows the same thing, lower the severity and say why. If nothing reaches the path today, call it hardening.
- **Category**: one of Security, Correctness, React, WordPress, Performance, Resources, Accessibility, Maintenance, Dependencies, Testing.
