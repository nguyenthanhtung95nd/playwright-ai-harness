# IMPLEMENT mode

Two branches: **A. Write a new test** (from a `pw_plan` case or a described flow) or **B. Refactor existing tests into POM**. Pick based on what the user asked; if unclear, ask.

---

## Branch A -- Write a new test

Test steps / plan case: as given by the user or handed off from `pw_plan`.

### A1. Review existing test suite

Read all existing spec files in `tests/` to understand what is already covered (avoid duplicating), which `test.describe` block and file the new test belongs in, the conventions in use (naming, grouping, `beforeEach` patterns), and whether page objects or fixtures from `playwright-utils/` (or equivalent) are available and relevant.

### A2. Get locators -- source available or not

**If application source code is reachable**, read it directly:

- Read the source of the relevant page/component and its children
- Identify locators using the project's priority order (see `pw_work`'s shared conventions)
- If no reliable user-visible locator exists, **add a `data-testid` to the application source**, then use `getByTestId`

**If application source is NOT reachable** (third-party site, deployed-only env, another team's app), use the Playwright CLI instead:

1. Confirm the URL with the user (default to `baseURL` from `playwright.config.ts`)
2. `npx playwright-cli open <baseURL>` to start a session
3. Walk the flow: `npx playwright-cli snapshot`, `click <ref>`, `fill <ref> "..."`, `select <ref> "..."`, `check <ref>`, `goto <url>` -- each returns a fresh snapshot with element refs
4. For every element the test touches, `npx playwright-cli generate-locator <ref>` to get a locator string, then validate it against the priority order -- a CSS-path or class-based result should be replaced with a semantic alternative read from the snapshot's role/name info if one exists
5. `npx playwright-cli close` when done
6. **Do not modify application source code** in this path -- if forced into a fragile locator, flag it explicitly to the user so they can request a `data-testid` from the app's owner

### A3. Write the test

- Place it in the correct file and `test.describe` block per existing structure
- Every test starts from the home page (`/`) -- reach inner pages via UI interactions, never a direct `goto` to an inner route
- Follow the project's locator/assertion/waiting/naming conventions
- No custom timeouts (see shared conventions in `SKILL.md`)
- One logical user flow per test
- **One statement per line** -- after writing, re-read the file and inline any vertically-stacked expressions (locator chains, `expect(...)` wraps, signatures) onto single lines; skip lines that are intentionally multi-line because their content has real newlines (template strings, multi-line array literals)

### A4. Ask to run or adjust

`AskUserQuestion` -- "Test is ready. What would you like to do next?" (Header: "Next step") -- **Run the test** / **Something else**. On "Something else", follow the custom instruction.

### A5. Run and debug

```bash
npx playwright test -g "<exact test title>" --trace on --retries 0
```

If it fails, use the trace-debug procedure in [VERIFY.md](VERIFY.md) ("Debug a failing test -- Trace") to find and apply the fix, then re-run. Repeat until it passes or the issue needs user input. If the failure is in the application rather than the test, explain it -- do not guess a workaround.

### A6. Confirm and finalize

`AskUserQuestion` -- "Test passed. Does it meet your expectations?" -- **Looks good** / **Needs changes**.

On **Looks good**: remove inline `//` comments (keep `test.describe` labels, titles, all code), collapse blank lines inside the test body, keep a single blank line only between top-level blocks. Then suggest VERIFY mode (audit) before SHIP.

On **Needs changes**: apply the feedback, re-run (A5), and loop.

---

## Branch B -- Refactor existing tests into POM

### B1. Ask scope

`AskUserQuestion` -- "Which tests should I refactor?" -- **All tests** (every spec under `tests/`) / **Specific test** (user names a file or title). Confirm the resolved scope.

### B2. Inventory the existing POM

Read everything under `playwright-utils/` (or the project's equivalent): page/component classes and their public methods, current fixtures and `PageManager` (if any), helpers available for reuse. If no POM exists yet, it gets bootstrapped in B5.

### B3. Analyze the tests in scope

For each spec in scope: read it end-to-end, read the source for each page/component it touches, group its steps **by the page they operate on** (every navigation marks a method boundary), and for each step group decide: **Reuse** an existing method, **Parametrize** a near-match, **Add** a method to an existing class, or **New page** class. Write this mapping down before touching code -- it prevents tiny one-off methods.

### B4. POM design rules (non-negotiable)

- One class per page/major component; kebab-case filename; PascalCase class name + `Page`/component suffix
- Constructor takes `Page` only -- no locator properties, no eager locator construction
- **Locators inline in methods** -- duplication across methods is fine, locators are lazy
- **No tiny methods** -- a method is a meaningful multi-step user task, never a single click/fill
- **Strict page boundaries** -- a method touches only its own page; navigation ends one method and starts the next on the next page's class
- **Validation at the start** -- if the method's first call auto-waits (click/fill/check/locator assertion), that's the start guard; otherwise gate non-auto-waiting code (`textContent`, `count`, `all`, `inputValue`, `allTextContents`) with an explicit `expect`/`waitFor` first
- **Validation at the end** -- when the last step is an action with no trailing assertion, add a stabilizing confirmation (`expect(page).toHaveURL(...)` after navigation; `toBeVisible`/`toHaveText`/`toContainText` for in-page changes)
- **Action vs validation methods** -- action methods may carry lightweight stabilizing assertions; the test's _goal_ assertion lives in a dedicated `expect*`-prefixed, parametrized validation method
- **Naming** -- camelCase, descriptive verb phrases, no abbreviations; validation methods start with `expect`
- **Reuse first** -- scan the class before adding a method; parametrize a near-match rather than duplicating; never write two methods differing only in a hardcoded value

### B5. Build or extend PageManager + the `pom` fixture

`PageManager` instantiates every page object **eagerly** in its constructor, exposed as camelCase properties; a single `pom` fixture is the only test entry point. Create `playwright-utils/fixtures/page-manager.ts` and `playwright-utils/fixtures/index.ts` if they don't exist. Adding a new page class means: create the file under `playwright-utils/pages/`, add the import + `readonly` property + constructor instantiation to `PageManager`.

### B6. Refactor the test files

Swap inline Playwright calls for `pom` calls. **Do not split tests, regroup describes, move steps between tests, or change `beforeEach` structure** -- only the test body changes. If a test still needs raw `page` for something not yet wrapped in a method, include both fixtures (`{ pom, page }`), but prefer adding the method.

### B7. Run and debug

```bash
npx playwright test <file-or-pattern> --trace on --retries 0
```

On failure, use the trace-debug procedure in [VERIFY.md](VERIFY.md). Common refactor-specific causes: a locator mismatch against the real source, a method doing work that belongs on the next page (split it), a missing start guard, or a missing end confirmation.

### B8. Confirm and finalize

`AskUserQuestion` -- "Refactor passes. Does it look right?" -- **Looks good** (strip `//` comments, collapse blank lines inside `test()` bodies, keep one blank line between top-level blocks, then suggest VERIFY mode) / **Needs changes** (apply feedback, re-run B7, loop).
