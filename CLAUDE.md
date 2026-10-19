# CLAUDE.md

Guidance for Claude Code when working in this repository.

## Who I Am

- Running a training session that teaches an AI-assisted E2E testing
  harness, using a small Angular app (Conduit) as the demo target.
- Primary interest is the harness itself (rules + skills that drive an AI
  agent through planning, writing, and shipping E2E tests), not Conduit's
  app features.

## What This Repo Is

1. **Conduit** — a Medium.com-style blogging app (Angular 18, standalone
   components). No real business data — safe to demo publicly.
2. **The actual deliverable: an AI-assisted Playwright E2E testing harness**
   — rules + skills that let an AI agent plan, write, verify, and ship E2E
   tests against its own rules, instead of a human catching violations by
   hand.

**Current state: the harness is not yet scaffolded in this repo.** There is
no `e2e/` folder, no `playwright.config.ts`, no `.husky/` hook, and
`package.json` has no Playwright dependency or `e2e`/`audit` scripts. What
exists already is the rules + skills layer under `.claude/` — the part that
drives `pw_setup` (INIT mode) to build the rest live, typically as part of
the training session itself. Don't assume `e2e/*` files exist; check first.

## Tech Stack

- **App:** Angular 18, standalone components (no NgModules), lazy-loaded
  routes in `src/app/app.routes.ts`.
- **Unit tests:** Karma/Jasmine (`npm test`) — no `*.spec.ts` under `src/`
  yet.
- **E2E tests:** not set up yet — see "What This Repo Is" above. Once
  scaffolded via `pw_setup` (INIT), it will be Playwright with config at
  `playwright.config.ts`.
- **Node:** `^18.13.0 || ^20.9.0`.

### Commands

```bash
npm install        # install deps
npm start          # ng serve, http://localhost:4200
npm run build      # production build -> dist/angular-conduit
npm test           # Karma/Jasmine unit tests
```

`npm run lint` is defined but not wired up (no `lint` architect target, no
ESLint config) — it will fail if invoked.

## Feature Development Pipeline

This repo has no general feature pipeline — all E2E test work goes through
three skills, in order:

```
No harness yet
    ↓
pw_setup (INIT)      → scaffold e2e/, playwright.config.ts, .husky/ hook
    ↓
Scenario / ticket
    ↓
pw_plan               → reviewed coverage plan saved to .claude/pw-plans/
    ↓
pw_work (IMPLEMENT)  → write or refactor the spec
    ↓
pw_work (VERIFY)     → review/debug the diff
    ↓
pw_work (SHIP)       → stage changes + draft commit message for a human
```

`pw_work` never runs `git commit` or `git push` itself, in any mode — SHIP
only drafts a message and leaves changes staged. `pw_setup` never commits on
the user's behalf either.

For anything outside the testing harness (app code, docs), there is no
fixed pipeline — just follow normal engineering judgment and the rules
below.

## Skills

| Skill | Purpose |
|-------|---------|
| `pw_setup` | INIT (scaffold the harness — needed first, since it doesn't exist yet) / SHARE / SYNC (reconcile drift between this project's `.claude/` and a user's `~/.claude/`). |
| `pw_plan` | Turn a scenario/ticket into a reviewed E2E coverage plan, saved to `.claude/pw-plans/`. Never skip plan mode. Only useful once the harness is scaffolded. |
| `pw_work` | IMPLEMENT (write/refactor a spec) / VERIFY (review or debug) / SHIP (stage + draft commit message only). Only useful once the harness is scaffolded. |

## Rules

- **`.claude/rules/testing-mindset.md` is the single source of truth** for
  how E2E tests must be written, and already exists even though no spec
  files do yet. Read it before writing or changing any spec. Eight rules,
  summarized:
  1. Pyramid gate — only test here what a unit/component test can't.
  2. Locators — `getByRole`/`getByLabel` first; never guess a selector.
  3. Waiting — no fixed waits; `waitForTimeout` is banned.
  4. Test data — create only through real app flows, never seed a DB/storage directly.
  5. Flakiness — a flaky test is a defect, not something to retry away (`retries: 0`).
  6. Assertions — one behavior per test; assert structurally, not on exact generated text.
  7. Independence — every test runs alone, in any order; page objects hold selectors/actions only, never assertions.
  8. Enforcement — anything checkable mechanically must be caught by a mechanical audit script or pre-commit hook, not review alone (both are scaffolded by `pw_setup` INIT, not present yet).

## App Architecture (quick reference)

- Routes guarded inline with `inject(UserService).isAuthenticated`
  (`/login`, `/register` block authenticated users; `/settings`, `/editor`
  require auth).
- `src/app/core/` — auth (`JwtService`, `UserService`), three HTTP
  interceptors in fixed order (`apiInterceptor` → `tokenInterceptor` →
  `errorInterceptor`), layout, shared models.
- Session restore on boot via `APP_INITIALIZER` in `app.config.ts`.
- `src/app/features/` — `article`, `profile`, `settings`, each with their
  own `components`/`pages`/`models`/`services`.
- `src/app/shared/` — cross-feature components, directives, pipes.
