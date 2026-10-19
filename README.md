# Playwright AI Harness — demo repository

This repository carries two things:

1. **The harness** — a set of rules and skills under `.claude/` that drive an AI
   agent through planning, writing, verifying, and shipping Playwright E2E
   tests against written, enforced rules rather than memory and review alone.
2. **Conduit** — a Medium.com-style social blogging app built with Angular,
   used as the demo target. It is a realistic single-page app with user
   authentication, full CRUD for articles and comments, tags, favorites,
   following other users, and paginated article feeds. There is no real
   business data in it, which is what makes it safe to demo.

## The harness

| Path                                 | What it is                                                           |
| ------------------------------------ | -------------------------------------------------------------------- |
| `.claude/rules/testing-mindset.md`   | Eight rules every E2E test here must follow, each with a stated why   |
| `.claude/skills/pw_setup/`           | INIT (scaffold the framework) / SHARE / SYNC                          |
| `.claude/skills/pw_plan/`            | Turn a ticket into a reviewed coverage plan before any test is written |
| `.claude/skills/pw_work/`            | IMPLEMENT / VERIFY (audit + debug) / SHIP                             |
| `docs/deck.html`                     | The session deck — open it in a browser                              |
| `docs/script.md`, `docs/script-vn.md` | Presenter script, English and Vietnamese                             |

The Playwright framework itself (`e2e/`, `playwright.config.ts`, the pre-commit
hook) is **not** committed here — `pw_setup` in INIT mode scaffolds it, which is
what the live demo does.

## Running the app

Requires Node.js `^18.13.0 || ^20.9.0`.

1. Install dependencies:

   ```bash
   npm install
   ```

2. Start the dev server:

   ```bash
   npm start
   ```

3. Open http://localhost:4200/ in your browser. The app reloads automatically
   when you change source files.

## Building

Run `npm run build` to build the project. The build artifacts are output to the
`dist/` directory.
