---
name: pw_plan
description: Plan Playwright E2E test coverage for a new ticket or parent scenario before any test code exists. Enters plan mode, researches existing tests and source code, brainstorms positive/edge/negative cases, and saves an approved coverage plan as a markdown artifact in `.claude/pw-plans/` for `pw_work` (IMPLEMENT mode) to consume. Use when the user hands over a new ticket, feature, or scenario and asks what should be tested, or says "plan coverage for X".
---

Build a comprehensive Playwright test coverage plan for a parent scenario provided by the user. The output is a markdown artifact in `.claude/pw-plans/` that `pw_work` (IMPLEMENT mode) consumes to generate concrete tests.

Parent scenario: $ARGUMENTS

**Never skip `AskUserQuestion` steps in this skill, even if told to work autonomously.**

## Instructions

### 1. Enter plan mode immediately

Before doing anything else, call the `EnterPlanMode` tool. All research in steps 2-5 happens in plan mode (read-only). No files are written until the user approves the plan in step 6.

If `$ARGUMENTS` is empty or too vague to plan against (e.g. less than ~5 words and no nouns identifying a feature/page/flow), use `AskUserQuestion` once to ask the user to describe the parent scenario before entering plan mode. Otherwise proceed.

### 2. Anchor on the parent scenario

Restate the parent scenario in one sentence to confirm understanding. Identify:

- **Actor / user role** -- the role performing the scenario (unauthenticated visitor, standard authenticated user, privileged/admin user, or any other role defined by the project)
- **Feature area** -- the part of the product the scenario lives in
- **Primary user goal** -- what success looks like for the actor

Discover the available roles from the project itself -- folders under `tests/`, project docs (`CLAUDE.md`, READMEs, rules files), and auth setup (`tests/auth.setup.ts` or equivalent). Do not assume a fixed role list.

If any of the three is ambiguous, make a reasonable assumption and flag it in the plan; do not block on clarification.

### 3. Review existing test suite

Read the spec files under `tests/` (or whichever directory the project uses) that are relevant to the actor and feature area:

- Note which tests already exist so the plan does **not** duplicate them
- Note conventions in use (file location, `test.describe` block, fixtures, helpers, page objects) and follow what the project already does
- Identify which existing file the new tests should land in, or whether a new spec file is warranted

If the project has a Playwright rules file (e.g. under `.claude/rules/`), read it and respect its conventions (naming style, locator priorities, assertion patterns).

### 4. Inspect application source code

For each page or component the parent scenario touches, read enough source to identify behavior worth testing: page/route/view files, child components, server-side endpoints or controllers (validation schemas, thrown errors, conditional branches), the data layer (uniqueness constraints, enums, nullability, status fields), and authorization checks. Adapt to whatever stack the project uses. The goal is to surface every observable branch a user could hit. **Do not infer behavior -- read the code.**

### 5. Brainstorm test cases across all categories

Produce a coverage plan that is MECE-style (mutually exclusive, collectively exhaustive) across three categories: happy path, edge cases, negative cases. Full prompts and sub-checklists for each category are in [CHECKLISTS.md](CHECKLISTS.md) -- work through every bullet there before finalizing the case list.

For each case, decide whether it is realistically testable in E2E (not a unit-test concern). Drop cases that require mocking internals or that the UI has no surface for. Mark anything ambiguous with a `?` so the user can confirm during review.

### 6. Present the coverage plan with `ExitPlanMode`

Call `ExitPlanMode` with a markdown plan body following the exact structure in [TEMPLATE.md](TEMPLATE.md): actor/feature/goal/suggested file/existing tests, assumptions, test cases grouped by category with user-behavior titles and P0/P1/P2 priority, out of scope, and open questions.

### 7. Save the artifact after approval

Once the user approves the plan (plan mode exits), write the same plan body to `.claude/pw-plans/<slug>.md`, where `<slug>` is a kebab-case slug derived from the parent scenario's short title. Create `.claude/pw-plans/` if it does not exist. If a file with that slug already exists, append a numeric suffix (`-2`, `-3`, ...) rather than overwriting.

### 8. Confirm and offer next step

After saving, use `AskUserQuestion`:

- Question: "Coverage plan saved to `.claude/pw-plans/<slug>.md`. What's next?"
- Header: "Next step"
- Option 1: label "Generate first test", description "Hand off to `pw_work` (IMPLEMENT mode) with the top P0 case from the plan"
- Option 2: label "Done", description "Stop here -- I'll generate tests later"

If the user picks **Generate first test**, hand off to `pw_work` (IMPLEMENT mode) using the first P0 case's title, pre-conditions, steps, and expectations as its input. If they pick **Done** or provide custom input, follow that instruction and stop.
