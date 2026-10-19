# SHARE mode

Copy `pw_*` skills from `~/.claude/skills/` into `<project>/.claude/skills/`, with conflict resolution, so teammates and CI (machines without your personal copies) get the same workflow.

## 1. Verify project context

Confirm cwd is a project root (`package.json` exists). Otherwise stop and say this mode must run from a project root.

## 2. Discover sources

```bash
ls -d ~/.claude/skills/pw_*/ 2>/dev/null
```

For each candidate, resolve symlinks (`readlink -f` or `realpath`) -- entries under `~/.claude/skills/` may symlink elsewhere. Use the resolved path as the source. Nothing found -> stop and report: "No `pw_*` skills found in `~/.claude/skills/`. Nothing to copy."

## 3. Validate integrity

For each candidate: must contain `SKILL.md` (else `[BROKEN: no SKILL.md]`, excluded); `SKILL.md` must start with `---` and have `name:` + `description:` frontmatter (else `[BROKEN: invalid frontmatter]`, excluded).

## 4. Classify each item

Compare each valid candidate against `<project>/.claude/skills/<skill-name>/`:

- **NEW** -- target does not exist
- **IDENTICAL** -- target exists, content matches (`diff -rq <source> <target>` empty)
- **DRIFTED** -- target exists, content differs

## 5. Show inventory and ask which to copy

Print a table with every item's status, including `[BROKEN]` entries (with reasons) for visibility. Then `AskUserQuestion` with multi-select: default-checked `NEW` and `DRIFTED`, default-unchecked `IDENTICAL`, `BROKEN` not selectable. Everything deselected -> stop and report nothing was copied.

## 6. Ensure target directory exists

```bash
mkdir -p .claude/skills
```

## 7. Copy and resolve conflicts

- **NEW** -> `cp -R <source-dir> .claude/skills/`
- **DRIFTED** -> per-item `AskUserQuestion`: **Overwrite** (remove project copy, `cp -R` from source) / **Show diff** (`git diff --no-index <project-dir> <source-dir>`, then re-prompt Overwrite/Skip) / **Skip** (leave untouched)
- **IDENTICAL** (if kept selected) -> skip with "already up to date"

## 8. Print summary

```
Copied:    <N> (<list>)
Skipped:   <N> (<list>)
Identical: <N>
Broken:    <N> (<list with reasons>)
```

## 9. Print precedence reminder

> **Heads up:** your personal copies in `~/.claude/skills/` will continue to override these project copies on **your** machine (Claude Code precedence: personal > project for skills). Teammates without the personal copies will use the project versions -- that's the point of committing them.

## Notes

- One-way only: never modify or delete files in `~/.claude/skills/`.
- Do not commit on the user's behalf -- leave files for review and staging.
- Do not touch any unrelated skill already in `.claude/skills/` -- only operate on items being copied.
- This mode copies whatever is currently in the user folder, as-is. Rule content for a from-scratch project comes from INIT mode's own self-contained reference files, not from here.
