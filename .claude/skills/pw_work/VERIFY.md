# VERIFY mode

Two jobs: **Audit** a just-written or just-changed test with fresh eyes, or **debug** a failing test. For debugging, pick the branch by evidence source: local trace (default), live interactive attach (when a static trace isn't enough), or CI artifacts (when it only failed in GitHub Actions).

Across every branch: classify the root cause into exactly one bucket before fixing anything -- **test bug**, **application bug**, **environment/config problem**, or **flake**. Never paper over an application bug or a flake with a longer timeout, a retry, or `force: true`. Say which bucket you placed it in and why -- that is the most important part of the report.

---

## Audit

You are running in a fresh session -- bring a clean, independent perspective. Do not assume a previous agent's choices were correct.

### 1. Identify the recent changes

`git status --short` and `git diff HEAD` define the **entire** audit scope.

- No uncommitted changes -> stop and say so; do not fall back to prior commits or a different scope.
- A diff with no Playwright-related files (no specs, page objects, fixtures, helpers) -> say so and exit.
- Otherwise, audit only the diff hunks -- never unchanged tests, untouched page objects, or unrelated files.

### 2. Establish project context

Read the project's Playwright rules in full (commonly `.claude/rules/playwright-scripting.md` and `playwright-architecture.md`, installed by `pw_setup` -- discover actual filenames if different). No rules files -> infer conventions from 2-3 representative specs and the test-utilities directory, and say the audit is based on inferred conventions.

Detect POM usage: a page-objects directory with at least one class **and** the changed spec importing a custom fixture (commonly `pom`) instead of `test` from `@playwright/test` directly. POM not in use is a valid setup, not a violation -- skip POM checks entirely in that case.

### 3. Read the touched application source

For each locator or interaction in the diff, read the frontend source that renders it (any framework, any layout) to judge whether the locator is correct, unique, and the most semantic option available.

### 4. Audit against the checklist

Cite the specific rule from the project's rules files for every finding -- never invent a rule; if the project's rules contradict a default below, the project's rules win.

- **Locators**: priority order followed; no CSS class-name selectors; a newly added `data-testid` actually exists in source and follows naming convention; action-method locators resolve to exactly one element; repeated UI patterns scoped to their container (no bare `.first()`); cross-page navigation asserts a captured source-page value on the destination; `name`/`hasText` preferred over `.filter()` unless a second filtration level is needed; locator text matches source exactly including the viewport-visible responsive variant; assertions scoped to the section a value actually belongs to.
- **Assertions**: locator (auto-retrying) assertions preferred over generic `expect(value).toBe(...)`; no `expect.soft(...)`; negative assertions wait for the DOM mutation first; `toHaveText`/`toContainText` preferred over `getByText(...).toBeVisible()`.
- **Waiting**: no `page.waitForTimeout(...)`; no redundant `waitFor`/`waitForURL` before an auto-waiting step; explicit waits only before non-auto-waiting methods (`all`, `count`, `textContent`, `inputValue`, `allTextContents`); no custom timeouts in a first draft.
- **Test structure**: names describe user behavior, not implementation; entry point follows project convention; one logical flow per test; `describe`/`beforeEach` placement matches the file's existing conventions.
- **Code style**: one statement per line; locator constants only when justified (repeated 3+ times, or opaque-and-used 2+ times); no intermediate-only constants; no re-declared locator under a variant name; `Values`/`Texts` suffix for extracted-text variables, element-type names reserved for locator variables; repeated clicks on the same element use `click({ clickCount: N })`.
- **POM rules** (only if POM is in use): constructor takes `Page` only; locators inline in methods; no tiny single-action methods; no method spans two pages; goal assertions live in a parametrized `expect*` method, not in an action method; start/end stabilizing guards present where needed; camelCase descriptive method names; no duplicate methods differing only by a hardcoded value; new page classes registered in `PageManager`; spec imports from the project's custom fixtures module.
- **Architecture**: spec lives in the correct directory for its actor/role/scenario; auth setup untouched unless intentionally targeted; stateless utilities live in the project's helpers location, not inlined.

### 5. Compose the report

One structured report, one table per severity, findings numbered sequentially across all tables so the user can reference them by ID:

```
## Audit summary
<one-sentence verdict>

## Findings -- Must-fix
| # | Location | Issue | Fix |

## Findings -- Should-fix
| # | Location | Issue | Fix |

## Suggestions
| # | Location | Improvement | Suggested change |

## What looks good
- <bullets reinforcing well-applied conventions>
```

Omit empty sections. Keep each cell to one line; combine rule + evidence as `<rule>: <evidence>` in **Issue** rather than spilling to a second line. Backtick code identifiers, paths, selectors. Strip directory prefixes in **Location** (filename:line only).

**Must-fix** = direct rule violation. **Should-fix** = the rules call it out as preferred but current code still works. **Suggestions** = stylistic, not codified.

### 6. Offer next step

`AskUserQuestion` -- "Audit complete. What would you like to do?" -- **Apply must-fix** / **Apply all** / **Discuss** (plus free-text finding numbers, e.g. "apply 1, 3, 5" -- honor those precisely). On an apply option, edit the spec (and page objects if POM is in use) for the selected findings, then re-run only the affected test (`npx playwright test -g "<title>" --retries 0`); debug via trace if it still fails. On **Discuss**, answer questions and only edit on explicit request.

---

## Debug a failing test -- Trace (default)

Use when a specific local test is failing and the cause isn't obvious yet.

1. **Confirm the test.** If not given an exact title or unique keyword, stop and ask -- never guess, never run the whole suite.
2. **Find it.** Read `playwright.config.ts` for `testDir` (fall back to `tests/`, `e2e/`, `playwright/`). Grep the test directory for the title, matching `test(`, `test.only(`, `test.skip(`, `test.describe(`. Multiple matches -> ask which one. No match -> say so and stop. Read the matched spec in full before running anything.
3. **Run it** with tracing forced on and retries off: `npx playwright test -g "<exact test title>" --trace on --retries 0` (add `--project=<name>` if the failure is project-specific). Passes -> tell the user it passed this run, note it may be flaky, and stop without opening a trace. Fails -> extract the error, stack frame, and trace zip path (`test-results/`, or find it: `ls -t test-results/**/trace.zip | head -5`).
4. **Analyze with the CLI, never the GUI** (`npx playwright trace`, not `show-trace` -- the GUI blocks the agent): `open <trace>` -> `actions` (look for `✗`, or `actions --grep="expect"`) -> `action <n>` -> `snapshot <n> --name after` (or `--name before`) -> `requests`/`request <id>` if data-related -> `console`/`errors` for page-side errors -> `close` when done.
5. **Report**: which test, what went wrong, evidence (error, expected vs received, snapshot), the failing `path:line`, and which bucket (test/app/env/flake) it falls in.
6. **Fix** only if it's clearly a test-side issue (wrong locator, wrong expected value, missing wait) -- apply it, re-run with `--retries 0`, then once more without `--trace on` to confirm a clean pass. If it's an application bug, ambiguous, or needs broader investigation, explain and propose next steps rather than guessing.

## Debug a failing test -- Live attach

Use when a static trace wasn't enough because the failure depends on live page state (race condition, timing, state-dependent behavior). Never edits the spec to insert `page.pause()` -- the CLI's `pause-at` does the same job without touching source.

1. **Gather inputs** via `AskUserQuestion`: test file path, test name (or unique keyword), the line to pause at, and an optional failure hint. Do not skip this even under an autonomous instruction.
2. **Validate**: the file exists and the line falls inside the named test; `npx playwright-cli list` shows no conflicting active session (offer `close-all` if one exists); detect the Playwright project from config if `projects:` is defined.
3. **Spawn** in the background: `PLAYWRIGHT_HTML_OPEN=never npx playwright test <file> -g "<name>" --project=<proj> --debug=cli` (omit `--project` if there's no `projects:` array).
4. **Wait for the session id** (`tw-XXXXXX`) by polling the captured log; confirm the paused test matches the target file (if a different project/dependency paused first, `attach` + `resume` past it and keep waiting).
5. **Attach**: `npx playwright-cli attach <session>`. Every subsequent call needs `-s=<session>`.
6. **Set the breakpoint**: `npx playwright-cli -s=<session> pause-at "<file>:<line>"`.
7. **Resume to it**: `npx playwright-cli -s=<session> resume`. `Session closed` means the line was unreachable from the run that happened -- re-check it.
8. **Investigate** with `-s=<session> --json` on every call: `snapshot` to see the page, `eval "el => el.getAttribute('disabled')" <ref>` to probe why something isn't actionable, `generate-locator <ref>` or `run-code "<ts>"` to rehearse a corrected call, `console`/`requests` to check for app errors or failed calls.
9. **Diagnose and report**: root cause (locator drift, missing wait, race, app bug), evidence (snapshot/eval/network excerpt), suggested fix with `path:line`. Ask before applying; apply via `Edit` only -- never leave a `page.pause()` or other debug artifact.
10. **Always clean up**, every exit path: `npx playwright-cli close-all`, then verify with `npx playwright-cli list` (should show no browsers); escalate to `kill-all` if not. No source restoration needed since the spec was never touched for the pause itself.
11. If a fix was applied, re-run without `--debug=cli` to confirm; if it still fails, report the new failure and ask rather than looping into another live session automatically.

## Debug a failing test -- CI (GitHub Actions)

Use when a test failed in CI and may not reproduce locally. Treat the CI trace as authoritative, not a local re-run.

1. **Confirm plumbing**: `gh --version`, `gh auth status`, and that the repo has a GitHub remote. Missing any of these -> stop and say what's missing; do not install or authenticate `gh` yourself.
2. **Identify the run**: from a run URL/ID, a workflow name, or (if nothing given) the most recent failed Playwright run on the current branch (`gh run list --branch "$(git branch --show-current)" --status failure`). Multiple candidates -> ask. Zero failed runs -> say so and stop.
3. **Read the failed log first**: `gh run view <run-id> --log-failed | tail -200` to get the failing test title(s), spec path, and first error before downloading anything. Multiple failures -> debug one first, say which, come back to the rest later.
4. **Download the `test-results` artifact**: `gh run download <run-id> --name test-results --dir .pw-ci-debug/<run-id>/test-results` (list artifact names first if the name differs). Empty or no `trace.zip` -> tracing is likely off in CI; tell the user to enable it (`trace: 'on-first-retry'` or `'retain-on-failure'`) and stop. `.pw-ci-debug/` is local and disposable -- mention adding it to `.gitignore` but do not commit that change yourself.
5. **Locate the right trace**: match the folder name to the test from step 3; prefer the last retry's trace if retries are present.
6. **Find the test source** the same way as the local trace branch (config `testDir`, grep, read in full).
7. **Analyze via the trace CLI** -- identical subcommands to the local branch (`open` -> `actions` -> `action <n>` -> `snapshot` -> `requests` -> `console`/`errors` -> `close`), with extra attention to network calls since CI's `BASE_URL` often differs from local.
8. **Classify**: test bug / application bug (don't weaken the test) / environment-config problem (wrong `BASE_URL`, missing seed data or auth state or env var -- the fix belongs in the workflow or config, not the spec) / flake (identify _why_, fix the underlying cause, never just add timeout/retry/`force: true`).
9. **Report**: run (workflow, run ID, URL), test (file + title), root cause bucket + one paragraph, evidence, failing `path:line`, proposed fix.
10. **Apply the fix** per the project's scripting rules (check `.claude/rules/playwright-scripting.md` first, then Cursor-style rules, then `CLAUDE.md`, then general Playwright best practice as a last resort) -- minimal and focused on the root cause. Application bugs and environment problems get written up, not masked in the spec.
11. **Verify locally only if plausible**; if the failure is CI-only (depends on CI's `BASE_URL` or seed data), say so explicitly rather than claiming "fixed" from an inconclusive local pass.
12. **Hand off** -- do not commit, do not push (see `SKILL.md`'s hard rule and `SHIP.md`). Tell the user what changed, whether it was verified locally and why or why not, and that the next CI run (after they push) is the real check.
