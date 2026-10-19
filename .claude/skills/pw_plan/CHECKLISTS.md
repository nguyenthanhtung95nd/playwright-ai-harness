# Coverage Brainstorm Checklists

Work through every bullet below for the scenario in play. Keep only cases observable through the UI.

## Happy path (positive)

- The canonical success flow described by the parent scenario
- Meaningful variants of success -- different valid inputs, optional flags, alternative valid paths through the same flow

## Edge cases

- **Input boundaries**: empty, whitespace-only, min/max length, unicode, leading/trailing spaces, case sensitivity, special characters
- **Numeric / quantity boundaries**: 0, 1, exactly at a threshold, just above a threshold, max
- **State boundaries**: empty list, exactly one item, list at pagination boundary, all-selected vs none-selected
- **Pre-existing state collisions**: duplicate identifier already exists, resource already in target state, repeated submission
- **Navigation edges**: back button mid-flow, refresh mid-flow, deep-link into a step that requires earlier state, closing and reopening a modal
- **Concurrency**: double-click submit, two tabs in parallel (only if testable through UI without mocks)

## Negative cases

- **Validation failures**: required field missing, malformed format, value too short/long, mismatched fields (e.g. confirm-password)
- **Authorization**: wrong role accessing a privileged route, unauthenticated user hitting an authenticated-only action, expired/invalid session
- **Access-level / visibility rules**: hidden/draft content not visible to non-privileged roles, private resources accessed by non-owners
- **External-service failures**: payment declined, third-party API error, webhook not yet received (only the user-visible side)
- **Not-found / 404**: bad identifier, deleted resource, mistyped URL
- **Forbidden mutations**: user editing a resource they don't own, non-eligible user attempting a privileged action
