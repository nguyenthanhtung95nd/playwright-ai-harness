---
name: pw_setup
description: One-time and cross-project Playwright setup -- scaffold a brand-new E2E framework with rules installed, share your personal pw_* skills into a project so teammates and CI get them too, or reconcile drift between a project's rules/skills and your user-level `~/.claude/`. Use when bootstrapping Playwright in a new repo, committing your Playwright skills for a team, or after editing a rule/skill on one side and wanting both sides back in sync.
---

Three modes, outside the per-ticket `pw_plan` -> `pw_work` lifecycle. Pick the mode from what the user asked; if unclear, ask.

**Never skip `AskUserQuestion` steps in this skill, even if told to work autonomously.**

## Modes

| Mode      | When                                                                                   | Details              |
| --------- | -------------------------------------------------------------------------------------- | -------------------- |
| **INIT**  | No Playwright framework exists yet in this project                                     | [INIT.md](INIT.md)   |
| **SHARE** | Your personal `pw_*` skills should be committed into this project for teammates/CI     | [SHARE.md](SHARE.md) |
| **SYNC**  | A rule or skill was edited on the project side or the user side and both need to match | [SYNC.md](SYNC.md)   |

## Shared conventions

- One-way only in SHARE: never modify or delete anything under `~/.claude/skills/`. SYNC is the only mode allowed to write to the user level, and only with the user's explicit direction per item.
- Directory copies never use `rm -rf`. Relocate the existing target to a timestamped path under `/tmp` first, then copy the source in fresh -- see [SYNC.md](SYNC.md) for the exact pattern.
- Do not commit on the user's behalf in any mode -- leave changes staged or unstaged for the user to review.
- Rule content for a brand-new project's `.claude/rules/` lives in this skill's own reference files, [RULE-ARCHITECTURE.md](RULE-ARCHITECTURE.md) and [RULE-SCRIPTING.md](RULE-SCRIPTING.md) -- self-contained, so INIT works on a machine with nothing else installed.
