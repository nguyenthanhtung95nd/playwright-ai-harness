# SYNC mode

Bidirectional sync of Playwright rules and `pw_*` skills between the current project and the user's `~/.claude/`. The user picks the winning direction for each drifted item -- no auto-merge.

- Rules: the **project** stores them as `<project>/.claude/rules/playwright-*.md`; the **user level** stores them as this skill's own `~/.claude/skills/pw_setup/RULE-<name>.md` reference files.
- Skills: directories `<project>/.claude/skills/pw_*/` <-> `~/.claude/skills/pw_*/`.

## 1. Verify project context

cwd must contain `package.json`. At least one of `.claude/rules/`, `.claude/skills/`, `~/.claude/skills/pw_setup/RULE-*.md`, or `~/.claude/skills/pw_*` must exist. Otherwise stop and say there is nothing to reconcile.

## 2. Discover candidates on both sides

```bash
# Project rules
ls .claude/rules/playwright-*.md 2>/dev/null

# Project skills
ls -d .claude/skills/pw_*/ 2>/dev/null

# User-level rule references (inside this skill, at user level)
ls ~/.claude/skills/pw_setup/RULE-*.md 2>/dev/null

# User skills
ls -d ~/.claude/skills/pw_*/ 2>/dev/null
```

Resolve symlinks for user-level `pw_*` directories (`readlink -f` or equivalent); confirm with the user before writing through a symlink.

## 3. Build the candidate set

Union of names seen on either side, partitioned into two groups:

- **Rules**: every `<name>` such that either `.claude/rules/playwright-<name>.md` exists OR `~/.claude/skills/pw_setup/RULE-<NAME>.md` exists.
- **Skills**: every `<name>` such that either `.claude/skills/pw_<name>/` exists OR `~/.claude/skills/pw_<name>/` exists.

## 4. Classify each item

- **IDENTICAL** -- present on both sides, content matches. No action.
- **DRIFTED** -- present on both sides, content differs. User picks direction.
- **MISSING_AT_USER** -- project has it, user does not. Implicit direction: project -> user.
- **MISSING_AT_PROJECT** -- user has it, project does not. Implicit direction: user -> project.

Comparison method: for rules, compare the project rule file against this skill's `RULE-<name>.md` content directly (both trimmed of leading/trailing whitespace) -- there is no marker-delimited block to extract since the rule lives as its own file in `pw_setup`, not embedded in a wrapper skill. For skills, `diff -rq <project-skill> <user-skill>` -- empty output means identical.

## 5. Show inventory

Print a table grouped by category, showing every state.

```
RULES (project <-> pw_setup reference)
  [DRIFTED]            playwright-architecture
  [IDENTICAL]          playwright-scripting
  [MISSING_AT_USER]    playwright-newrule          (project has it, user doesn't)
  [MISSING_AT_PROJECT] playwright-otherrule        (user has it, project doesn't)

SKILLS (project <-> user)
  [DRIFTED]            pw_work
  [MISSING_AT_USER]    pw_plan
  [MISSING_AT_PROJECT] pw_something_else
```

## 6. Ask which items to sync

**Skip this step entirely** when the actionable set is a single item -- exactly one `DRIFTED` and zero `MISSING_AT_USER`/`MISSING_AT_PROJECT`. Jump straight to step 7 for that one item.

Otherwise, `AskUserQuestion` with multi-select: default-checked `DRIFTED`, `MISSING_AT_USER`, `MISSING_AT_PROJECT`; default-unchecked `IDENTICAL`. For each `DRIFTED` option, the description must end with " (you'll pick the winning side next)". For `MISSING_AT_USER`/`MISSING_AT_PROJECT`, the description must state the implicit direction. Everything deselected -> stop and report nothing was synced.

## 7. Resolve direction per drifted item

For each selected `DRIFTED` item, `AskUserQuestion`: **Use project version** (project -> user) / **Use user version** (user -> project) / **Show diff** (`git diff --no-index <user-side> <project-side>`, then re-prompt minus Show diff) / **Skip**. For `MISSING_AT_USER`/`MISSING_AT_PROJECT`, direction is implicit -- no question, just confirm in the summary.

## 8. Apply updates -- override pattern (no `rm -rf`)

Never `rm -rf`. Relocate the existing target with `mv` to a timestamped `/tmp` path first, then copy the source in fresh:

```bash
TS=$(date +%s)
[ -e <target> ] && mv <target> /tmp/<basename>-<role>-$TS
cp -R <source> <target_parent>/
```

`<role>` is `project` or `user`, recording which side was overwritten. The `/tmp` copy is OS-cleaned and never registers as a skill (outside `.claude/skills/`). Tell the user where the backup landed.

### Rules: project -> user

Covers `MISSING_AT_USER` and `DRIFTED` where the user picked project. Write the project rule file's content verbatim to `~/.claude/skills/pw_setup/RULE-<NAME>.md` (creating `~/.claude/skills/pw_setup/` if this is the very first sync from a project that has `pw_setup` only locally -- ask before creating a new user-level `pw_setup` skill from scratch this way).

### Rules: user -> project

Covers `MISSING_AT_PROJECT` and `DRIFTED` where the user picked user. `mkdir -p .claude/rules` if needed, then write `~/.claude/skills/pw_setup/RULE-<NAME>.md`'s content verbatim to `.claude/rules/playwright-<name>.md` (overwrite directly via `Write` -- text files are safe to overwrite).

### Skills: project -> user

```bash
TS=$(date +%s)
[ -e ~/.claude/skills/<name> ] && mv ~/.claude/skills/<name> /tmp/<name>-user-$TS
cp -R .claude/skills/<name> ~/.claude/skills/
```

If the user-level entry was a symlink, write to the resolved real path instead, only after confirming.

### Skills: user -> project

```bash
TS=$(date +%s)
mkdir -p .claude/skills
[ -e .claude/skills/<name> ] && mv .claude/skills/<name> /tmp/<name>-project-$TS
cp -R ~/.claude/skills/<name> .claude/skills/
```

## 9. Print summary

```
Rules:
  Project -> User:    <N> (<list>)
  User -> Project:    <N> (<list>)

Skills:
  Project -> User:    <N> (<list>)
  User -> Project:    <N> (<list>)

Skipped:             <N>
Identical:           <N>

Backups (in /tmp, OS-cleaned):
  <list of /tmp/<name>-<role>-<ts> paths created during this run>
```

## 10. Print follow-up reminder

> **Next steps:**
>
> - To propagate updates to other projects, run `pw_setup` (INIT mode) or SHARE mode in each one.
> - Skills precedence: on **your** machine, the user-level `pw_*` always overrides any project-level copy of the same name (Claude Code precedence: personal > project). Pushing into a project via SHARE does **not** change this -- it exists to give teammates and CI the same skill via the project copy.
> - Backups in `/tmp/` are not auto-deleted by this mode. The OS clears them on reboot or after a few days; remove sooner if disk space matters.

## Notes

- Bidirectional, but direction is always the user's call per drifted item -- no auto-merge.
- Removal is out of scope. If a `pw_*` or rule exists on only one side and the user wants to delete rather than copy, they do so manually.
- Prefer a clean working tree when possible -- easier to roll back project-side changes via git.
