# SHIP mode

The last gate before a human commits, held to the same safety standard as every other mode in this workflow: **stage and summarize, never commit, never push.**

## Gate

Before doing anything else, confirm VERIFY is clean:

- If an audit ran and left **Must-fix** findings unresolved, stop. Send the work back to IMPLEMENT or VERIFY and say which findings are blocking.
- If no audit has run yet for this change, run one now (VERIFY mode, Audit branch) before continuing.

A Must-fix finding is a hard stop -- fix it and re-verify before coming back here.

## Steps

1. **Confirm the diff matches intent.** `git status` and `git diff` against the scope of the ticket/plan. Flag anything in the diff that wasn't part of the plan -- do not ship unrelated changes silently.
2. **Final verify pass.** Re-run what the project defines for this: typecheck (`npx tsc --noEmit` if the project is TypeScript), its lint command if one exists, and the affected spec(s) with `--retries 0`. All green before continuing.
3. **Stage only the relevant files.** `git add` the files that belong to this change -- never a blanket `git add -A` without reviewing what it picked up.
4. **Draft a commit message**, matching the repository's existing style (`git log` for recent examples). Do not create the commit.
5. **Write a short Work Summary**:
   ```
   ## Work Summary
   **Problem** -- one sentence.
   **What Changed** -- files + what changed in each.
   **How It Was Verified** -- audit result, debug bucket if applicable, test run output (pass/fail counts, run time).
   **Follow-ups** (optional) -- anything deferred or still open.
   ```
6. **Hand off.** Tell the user the change is staged and summarized, give them the drafted commit message, and stop. They run `git commit` (and `git push`) themselves.

## Hard rule

Never run `git commit`, `git push`, or `git rebase --continue` in this mode -- not even if the user says "yes, commit it" in response to a question this skill asks. If a workflow seems to require committing to proceed, stop and report that instead of doing it.
