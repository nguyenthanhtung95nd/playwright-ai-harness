# E2E Testing Mindset -- conduit

Read this before writing or changing any end-to-end test in this repo.
Automated tests exist for one reason: fast, trustworthy feedback. Every rule
below protects that, not style for its own sake.

## 1. Pyramid gate

Before adding a test here, ask: could a unit or component test prove the same
thing cheaper? E2E is for user journeys that cross real boundaries (routing,
auth, API, storage) -- not for input validation, formatting, or pure
component logic. If a lower layer could catch it, it does not belong here.

**Why:** a unit test runs in milliseconds and costs nothing to maintain when
the UI changes; an E2E test is the slowest, most expensive layer this project
has. Paying that cost without a reason is the single biggest tax on fast
feedback. Skipping this gate is also how a suite drifts into the ice-cream
cone antipattern -- many slow UI tests and almost no fast ones -- where every
small change waits on the whole browser suite to find out if it broke
anything.

## 2. Locators

- `getByRole` / `getByLabel` first, always. An accessible name that is
  missing or ambiguous is a defect in the app, not a reason to fall back to a
  CSS selector.
- Never guess a locator. If the accessible name is unclear, stop and flag it.
- No XPath, no brittle CSS chains, no auto-generated test-id as a first
  choice -- if one is needed it is a last resort, not a default.

**Why:** a role/label locator only matches when the name, role, and state are
actually correct for assistive technology -- the locator strategy doubles as
a free, continuous accessibility check. A CSS selector or a test-id can point
at the right pixel while hiding a broken or missing accessible name; picking
by role/label makes that defect impossible to paper over. The same discipline
applies to uniqueness: a locator used for an action must resolve to exactly
one element, and a locator inside a repeated pattern (a list, a card grid)
must be scoped to its own container -- an unscoped `.first()` can silently
pass by matching the wrong instance of the same text.

## 3. Waiting

- No fixed waits, ever. `waitForTimeout` is banned outright.
- Use web-first assertions (`expect(locator).toBeVisible()`, `toHaveText()`,
  etc.) and condition-based waits (`waitForResponse`, `toPass`).
- An article, a comment, a follow state -- all of these update through a real
  network round trip. Wait for the state, never for a clock.

**Why:** a fixed wait encodes one machine's timing, not the product's actual
behavior. A delay long enough to be safe on a loaded CI runner is dead time on
every fast local run; a delay tuned for a fast run is a flake waiting to
happen the first time the network is slow. Waiting for the state itself
removes timing as a variable entirely, instead of gambling on it.

## 4. Test data

- Create everything through the app's own flows: register through the UI,
  publish an article through the editor, favorite through the button.
- Never insert rows directly into a database or local storage to fake state.
  If the app's own create-path is broken, a test that bypasses it will never
  catch that.
- Give every run its own unique identity (timestamped username/email/slug)
  so tests stay independent and parallel-safe.

**Why:** data created through the app's own write path is automatically
protected by every validation rule and schema change that path enforces; a
row inserted directly bypasses all of it and rots the moment the real flow
changes underneath it. The app's create-path breaking silently is exactly the
defect class this rule exists to catch -- bypassing it to save setup time
removes the one check that would have caught it.

## 5. Flakiness

- A test that fails once and passes on retry is not "flaky," it is a defect
  report about a race condition. `retries: 0` locally and in CI.
- Never silence a flake with a broader locator, a longer timeout, or a retry
  wrapper. Find the race, fix the wait.

**Why:** a red result that gets quietly retried into green poisons feedback
for every change that lands behind it -- nobody can tell a real regression
from background noise anymore, and whoever sees the failure first inherits
the job of proving which one it was. Before touching anything, name which
bucket the failure is actually in: a race with async product behavior, test
interdependence, dynamic data, or environment startup ordering. Naming the
bucket is what turns a guess into a fix; patching the symptom (a longer
timeout, a broader locator, a retry wrapper) leaves the same race ready to
fire again on the next slow run.

## 6. Assertions

- One behavior per test. The test name states the intent; a failure should
  name the defect without needing to read the test body.
- Assert structurally where content is generated or timestamped (e.g. "an
  article now appears in the feed"), never on exact rendered text that can
  legitimately vary.

**Why:** the test suite is the only documentation of this product's behavior
that future readers will actually trust, because code and comments drift but
a test that still passes is still true. A name and a set of steps should
teach what the product does without anyone opening the implementation.
Asserting on exact generated or timestamped text makes a test fail for a
reason that has nothing to do with correctness -- it breaks on every run that
happens to land at a different second, which trains people to ignore red
results instead of trusting them.

## 7. Independence

- Every test must be runnable alone and in any order. No test may depend on
  state left behind by another.
- Page objects hold selectors and actions only -- zero assertions inside a
  page object. Specs assert; pages expose state.

**Why:** a chained test produces a chained error -- when test B silently
depends on state test A left behind, a failure in A reports as a confusing
failure in B, and the two can never safely run in parallel. Keeping
assertions out of page objects is the same discipline at a different layer:
fix a UI change exactly once, in the page object, instead of hunting the same
broken assertion across every spec that happened to inline it.

## 8. Enforcement

A rule that is only ever checked by memory is not a rule, it is a hope. Any
rule in this file that can be checked mechanically (banned APIs, forbidden
patterns) should be caught by an automated audit or a pre-commit hook, not
left to code review alone.

**Why:** `e2e/audit.mjs` today mechanically checks exactly two of the eight
rules above -- Rule 3 (Waiting) and Rule 4 (Test data) -- and nothing else
yet. That gap is the backlog, not proof the other six rules matter less: a
rule with no mechanical check still has to be enforced by a reviewer every
time, by hand, which is exactly the failure mode this section exists to
close. When an audit does run, it should cite the specific rule a change
violates rather than an opinion -- a finding with no rule behind it does not
belong in this file's enforcement story.
