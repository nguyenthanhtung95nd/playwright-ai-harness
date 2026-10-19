# Coverage Plan Output Template

Use a user-behavior naming style for test titles (subject + can/cannot/sees + observable behavior -- e.g. `<Role> can <do X>`, `<Role> sees error when <condition>`). Match the exact style already used in the project's existing tests if one is established. Steps and expectations describe what the user does and observes, not selectors or implementation. Priorities: `P0` = must-have for parent scenario, `P1` = important, `P2` = nice-to-have.

```markdown
# Coverage Plan: <parent scenario short title>

**Actor:** <role>
**Feature area:** <area>
**Primary goal:** <one sentence>
**Suggested file:** `<path/to/spec/file>.spec.ts` (new | existing)
**Existing related tests:** <list of test titles already in the suite, or "none">

## Assumptions

- <bullet for each assumption made about ambiguous parts of the scenario>

## Test cases

### Happy path

1. **<test title in user-behavior style>** -- <one-line summary> _(P0)_
   - Pre: <auth state, test data needed>
   - Steps: <bullets, user-level -- no selectors>
   - Expect: <observable outcomes -- UI state, URL, persisted data visible in UI>

### Edge cases

2. **<test title>** -- <summary> _(P1)_
   - Pre / Steps / Expect ...

### Negative cases

3. **<test title>** -- <summary> _(P1)_
   - Pre / Steps / Expect ...

## Out of scope

- <cases considered but dropped, with one-line reason -- e.g. "covered by unit test", "no UI surface", "requires mocking external service">

## Open questions

- <anything marked `?` during brainstorming that the user should confirm>
```
