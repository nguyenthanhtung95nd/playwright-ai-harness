---
name: pw_work
description: Carry a Playwright E2E ticket from a coverage plan through to a shippable change, in three modes -- IMPLEMENT (write a new test or refactor existing ones into Page Object Model), VERIFY (audit the diff with fresh eyes, or debug a failing test via trace, live attach, or CI artifacts), SHIP (stage and summarize for a human to commit, never commit or push autonomously). Use after `pw_plan` has produced a coverage plan, or directly when the user asks to write, fix, debug, audit, or ship a Playwright test.
---

State which mode you are in before acting. Pick the mode from what the user asked; if unclear, ask.

**Never skip `AskUserQuestion` steps in this skill, even if told to work autonomously.**

## Modes

| Mode          | When                                                                                                                | Details                      |
| ------------- | ------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| **IMPLEMENT** | Turn a plan (from `pw_plan`) or a described flow into a new test, or refactor existing tests into Page Object Model | [IMPLEMENT.md](IMPLEMENT.md) |
| **VERIFY**    | A test was just written or changed and needs a fresh review, or a test is failing and the cause is unclear          | [VERIFY.md](VERIFY.md)       |
| **SHIP**      | VERIFY is clean and the change is ready to hand to a human for commit                                               | [SHIP.md](SHIP.md)           |

The natural flow is IMPLEMENT -> VERIFY -> SHIP, but any mode can be entered directly (e.g. "debug this failing test" goes straight to VERIFY's debug branch).

## Conventions that hold in every mode

- **Locator priority**: `getByRole` > `getByLabel` > `getByText` > `getByPlaceholder` > `getByTestId` > CSS (stable structural tags only, never style classes). If the project has `.claude/rules/playwright-scripting.md` (installed by `pw_setup`), its conventions win over this default -- read it first.
- **No custom timeouts** in a first draft -- no `test.setTimeout()`, no `{ timeout: ... }` on assertions or waits. A custom timeout is only ever added as a diagnosed debugging fix, never preemptively.
- **No `page.waitForTimeout(...)`, ever.** Use auto-waiting actions and locator assertions instead.
- **`retries: 0`** when running or re-running a test from this skill -- a pass-on-retry is a defect signal, not a reason to move on.
- **Never weaken a test to make an application bug disappear.** If the root cause is in the app, report it; do not adjust the assertion.
- **Never guess a locator.** If no reliable semantic locator exists, the fix is a `data-testid` added to the application source (when source is reachable) -- not a CSS class selector.

## Hard rule -- no autonomous commits

Never run `git commit`, `git push`, or anything that publishes a commit, in any mode, even if the user says yes to a commit prompt. SHIP stages and drafts a message; a human runs the commit themselves. See [SHIP.md](SHIP.md).
