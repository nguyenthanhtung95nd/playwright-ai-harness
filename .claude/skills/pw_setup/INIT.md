# INIT mode

Initialize a Playwright E2E framework from scratch in the current project.

## Conventions the generated framework follows

```
e2e/
  audit.mjs                       # Mechanical rules-aware audit (only if opted in at 1.4)
  auth.setup.ts                   # Auth state persistence (only if roles requested)
  <role>/                         # One subfolder per auth role (e.g. guest, admin)
    smoke.spec.ts                 # Generated sample smoke test
playwright-utils/
  .auth/                          # storageState files -- gitignored
  fixtures/
    index.ts                      # Extended test + expect re-export (skeleton)
  helpers/                        # Stateless utilities
  pages/                          # Page objects (empty -- created on demand)
  types/
    env.d.ts                      # process.env typing
.env.test.<env>                   # Per-environment vars -- gitignored
playwright.config.ts              # Single Chromium browser, one project per role
```

Key conventions: single browser (Desktop Chrome, no Firefox/WebKit); env files named `.env.test.<env>` loaded via `dotenv` based on `TEST_ENV` (default `local`); one project per role in `playwright.config.ts`, authenticated roles getting `storageState` + `dependencies: ['auth-setup']`; storage state lives in `playwright-utils/.auth/<role>.json`.

## 1. Gather requirements

Use `AskUserQuestion` and free-form chat, in this order:

### 1.1 Environments

Question: "How many test environments do you need, and what should they be called? Provide a baseURL for each." Options: `Just local`, `local + qa`, `local + qa + production` (plus free-form "Other"). Follow up in chat for any additional baseURLs. The first env listed becomes the default.

### 1.2 Auth roles

Question: "What auth roles do you need? Each authenticated role gets an entry in `auth.setup.ts` plus its own project with storageState. 'guest' is unauthenticated." Options: `guest only`, `guest + user`, `user + admin`, `guest + user + admin` (plus "Other"). If only `guest` is in the final list, skip `auth.setup.ts` and the `auth-setup` project.

### 1.3 Login flow details (only if any authenticated roles)

Question: "How should I figure out the login flow for `auth.setup.ts`? I need: link text to the login page from `/`, form field labels for email and password, and the URL each role lands on after login." Options: **Explore the source code** (search `src/app/login`, `app/login`, `src/pages/login`, `Login*` components, or routes behind a "Log In"/"Sign In" link; fall back to placeholders if nothing reliable turns up), **Generate placeholders** (generic `<LOGIN_LINK_TEXT>`/`<EMAIL_LABEL>`/`<PASSWORD_LABEL>`/`<POST_LOGIN_PATH>` with a `// TODO` marker), or **I'll provide the details** (parse link text, field labels, post-login path from the user's answer). Do not block on this -- pick a path and continue.

### 1.4 Optional extras

Three yes/no questions: a `webServer` block in `playwright.config.ts` to auto-start the dev server in CI (if yes, ask the start command and URL); whether to copy the architecture/scripting rules into `.claude/rules/` (content comes from this skill's own [RULE-ARCHITECTURE.md](RULE-ARCHITECTURE.md) / [RULE-SCRIPTING.md](RULE-SCRIPTING.md) -- no source path to ask for); and whether to wire a mechanical rules audit into a pre-commit hook (Husky + lint-staged) so a banned pattern (e.g. a fixed wait) is rejected before it reaches the repo, not just caught in review. If this project already has a pre-commit hook or an audit script, skip this question and say so instead.

## 2. Run the Playwright installer

```bash
npm init playwright@latest -- --quiet --browser=chromium --install-deps
```

`--quiet` skips the wizard; `--browser=chromium` matches the single-browser convention; `--install-deps` installs browsers and system deps; TypeScript + `e2e/` + no GitHub Actions workflow are the quiet-mode defaults. If the install fails or stalls, ask the user to run it interactively.

## 3. Clean up installer defaults

```bash
rm -r tests-examples 2>/dev/null
rm e2e/example.spec.ts 2>/dev/null
```

## 4. Install supporting dependencies

Ask the user to run `npm install -D dotenv`.

## 5. Generate `playwright.config.ts`

```typescript
import { defineConfig, devices } from "@playwright/test";
import dotenv from "dotenv";

const testEnv = process.env.TEST_ENV ?? "<DEFAULT_ENV>";
process.env.TEST_ENV = testEnv;
dotenv.config({ path: `.env.test.${testEnv}`, quiet: true });

const isCI = !!process.env.CI;

export default defineConfig({
  testDir: "./e2e",
  fullyParallel: false,
  retries: isCI ? 2 : 0,
  workers: isCI ? 1 : undefined,
  reporter: isCI ? [["html", { open: "never" }], ["github"]] : [["html", { open: "never" }]],
  timeout: 60000,
  expect: { timeout: 10000 },
  use: {
    baseURL: process.env.BASE_URL,
    trace: "on-first-retry",
  },
  projects: [
    // Include only if any authenticated roles exist:
    { name: "auth-setup", testMatch: "auth.setup.ts", use: { ...devices["Desktop Chrome"] } },
    // One entry per role. Unauthenticated roles (e.g. guest) omit storageState + dependencies:
    {
      name: "<role>",
      testDir: "./e2e/<role>",
      use: { ...devices["Desktop Chrome"], storageState: "playwright-utils/.auth/<role>.json" },
      dependencies: ["auth-setup"],
    },
  ],
  // Include only if the user opted into a webServer block:
  // webServer: { command: '<USER_DEV_COMMAND>', url: '<USER_DEV_URL>', reuseExistingServer: !process.env.CI, timeout: 120_000 },
});
```

Replace `<DEFAULT_ENV>` with the first env name. One project entry per role.

## 6. Update `.gitignore`

```
# Playwright
playwright-utils/.auth/
test-results/
playwright-report/
playwright/.cache/

# Local env files (keep .env.test.example committed)
.env
.env.local
.env.test.*
!.env.test.example
```

## 7. Create directory structure

```bash
mkdir -p playwright-utils/{fixtures,helpers,pages,types,.auth}
mkdir -p e2e/<role>  # one per role from 1.2
```

## 8. Generate `e2e/auth.setup.ts`

Skip if no authenticated roles. Otherwise one `setup(...)` block per authenticated role:

```typescript
import { test as setup } from '@playwright/test'

const <role>File = 'playwright-utils/.auth/<role>.json'

setup('authenticate as <role>', async ({ page }) => {
  await page.goto('/')
  await page.getByRole('link', { name: '<LOGIN_LINK_TEXT>' }).click()
  await page.getByRole('textbox', { name: '<EMAIL_LABEL>' }).fill(process.env.<ROLE>_EMAIL!)
  await page.getByLabel('<PASSWORD_LABEL>').fill(process.env.<ROLE>_PASSWORD!)
  await page.getByRole('button', { name: 'Login', exact: true }).click()
  await page.waitForURL('**<POST_LOGIN_PATH>')
  await page.context().storageState({ path: <role>File })
})
```

Partial details from 1.3 -> leave placeholders with a single `// TODO: confirm selectors` comment above the block (marks user-action-required, not narration -- allowed).

## 9. Generate `playwright-utils/types/env.d.ts`

```typescript
export {}

declare global {
  namespace NodeJS {
    interface ProcessEnv {
      TEST_ENV?: '<env1>' | '<env2>' | ...
      BASE_URL?: string
      CI?: string
      // One pair per authenticated role:
      <ROLE>_EMAIL?: string
      <ROLE>_PASSWORD?: string
    }
  }
}
```

`declare global` is required because `export {}` makes this a module -- without it, TypeScript treats the `NodeJS.ProcessEnv` augmentation as local and `process.env.<ROLE>_EMAIL` won't be typed. The default Playwright `tsconfig.json` covers `**/*.ts`, so this file is picked up automatically.

## 10. Generate `playwright-utils/fixtures/index.ts`

```typescript
import { test as base, expect } from "@playwright/test";

type Fixtures = {
  // Add fixtures here as page objects and helpers are built.
};

export const test = base.extend<Fixtures>({});

export { expect };
```

## 11. Generate sample smoke tests

One `e2e/<role>/smoke.spec.ts` per role.

**Guest smoke:**

```typescript
import { test, expect } from "@playwright/test";

test.describe("Guest smoke", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/");
  });

  test("Home page loads", async ({ page }) => {
    await expect(page).toHaveURL("/");
  });
});
```

**Authenticated role smoke:**

```typescript
import { test, expect } from "@playwright/test";

test.describe("<Role> smoke", () => {
  test("Authenticated session is active", async ({ page }) => {
    await page.goto("/");
    // TODO: replace with a real authenticated-only assertion
  });
});
```

## 12. Update `package.json` scripts

Merge into existing `scripts` (don't overwrite):

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui"
  }
}
```

If more than one environment was defined, add `"test:e2e:<env>": "TEST_ENV=<env> playwright test"` per non-default env.

## 13. Wire a mechanical rules audit and pre-commit hook (only if the user opted in at 1.4)

```bash
npm install -D husky lint-staged
npx husky install
```

Read this project's actual rule file (`.claude/rules/testing-mindset.md`, or whatever the equivalent is named here) in full and list every numbered rule in it -- do not work from memory or from a generic rule set. Then use `AskUserQuestion` (multi-select) to ask: "Which of these rules should `e2e/audit.mjs` check mechanically?" Pre-check whichever rules map cleanly onto a reliable, low-false-positive text pattern; leave the rest unchecked but still listed as options. A rule that needs judgment to evaluate (e.g. "one behavior per test," "could a lower layer prove this cheaper") has no reliable pattern -- say so explicitly rather than forcing a check onto it, and note that it stays review-only, the same gap this project's own rule file already documents about itself under its Enforcement rule.

Known reliable patterns to offer when the corresponding rule is present (match by the rule's actual meaning in the file just read, not by number -- numbering varies per project):

| Rule meaning | Pattern | Message |
|---|---|---|
| No fixed waits | `waitForTimeout\s*\(` | Fixed wait found. Replace with a web-first assertion or a condition-based wait (waitForResponse, toPass). |
| No direct data seeding | `\b(INSERT INTO\|prisma\.\w+\.create\|db\.collection\()` | Looks like direct data seeding. Create test data through the app's own UI flow instead. |
| Role/label locators only | `page\.locator\(\s*['"](\.|#|//)` | Looks like a CSS/XPath locator. Use getByRole/getByLabel instead, or flag the missing accessible name as an app defect. |
| Zero tolerance for retries | `retries:\s*[1-9]` | Non-zero retries found. A flaky pass is a defect report, not something to smooth over. |

If the project's rule file has a rule with no row above, ask the user directly whether they know a reliable pattern for it before skipping it.

For each rule the user selected, add one `CHECKS` entry using the rule's real number and name from the file just read. Write the resulting `e2e/audit.mjs`:

```javascript
#!/usr/bin/env node
// Rules-aware audit for e2e/**.spec.ts -- checks specs against this
// project's rule file. Exits non-zero if any rule is violated, so this
// script can be run by hand or wired into a git hook / CI step.

import { readFileSync, readdirSync, statSync } from "node:fs";
import { join, relative } from "node:path";

const ROOT = new URL("..", import.meta.url).pathname.replace(/^\/([A-Za-z]:)/, "$1");
const E2E_DIR = join(ROOT, "e2e");

const CHECKS = [
  // One entry per rule the user selected above, e.g.:
  // {
  //   rule: "Rule 3 -- Waiting",
  //   pattern: /waitForTimeout\s*\(/g,
  //   message: "Fixed wait found. Replace with a web-first assertion or a condition-based wait (waitForResponse, toPass).",
  // },
];

function walk(dir) {
  const out = [];
  for (const entry of readdirSync(dir)) {
    const full = join(dir, entry);
    const stat = statSync(full);
    if (stat.isDirectory()) out.push(...walk(full));
    else if (entry.endsWith(".spec.ts")) out.push(full);
  }
  return out;
}

const specFiles = walk(E2E_DIR);
const violations = [];

for (const file of specFiles) {
  const text = readFileSync(file, "utf8");
  const lines = text.split("\n");
  for (const check of CHECKS) {
    const re = new RegExp(check.pattern.source, check.pattern.flags);
    let match;
    while ((match = re.exec(text)) !== null) {
      const upToMatch = text.slice(0, match.index);
      const lineNo = upToMatch.split("\n").length;
      violations.push({
        file: relative(ROOT, file),
        line: lineNo,
        code: lines[lineNo - 1].trim(),
        rule: check.rule,
        message: check.message,
      });
    }
  }
}

if (violations.length === 0) {
  console.log(`AUDIT: ${specFiles.length} spec file(s) checked, 0 violations.`);
  process.exit(0);
}

console.log(`AUDIT: ${specFiles.length} spec file(s) checked, ${violations.length} violation(s):\n`);
for (const v of violations) {
  console.log(`  ${v.file}:${v.line}  [${v.rule}]`);
  console.log(`    ${v.code}`);
  console.log(`    -> ${v.message}\n`);
}
process.exitCode = 1;
```

Write `.husky/pre-commit`:

```sh
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
node e2e/audit.mjs
```

Merge into `package.json` (don't overwrite existing `scripts` or `lint-staged` keys):

```json
{
  "scripts": { "prepare": "husky install" },
  "lint-staged": { "*.{ts,html,css,json,md}": "prettier --write" }
}
```

If `prettier` is not already a dependency, run `npm install -D prettier` first. In the final summary (step 15), list which rules got a mechanical check and which were left review-only, by name -- the gap should be visible, not silent.

## 14. Write the architecture/scripting rules (only if the user opted in at 1.4)

```bash
mkdir -p .claude/rules
```

Write [RULE-ARCHITECTURE.md](RULE-ARCHITECTURE.md)'s content to `.claude/rules/playwright-architecture.md`, and [RULE-SCRIPTING.md](RULE-SCRIPTING.md)'s content to `.claude/rules/playwright-scripting.md`, verbatim. If a target file already exists, ask **Overwrite / Skip** before overwriting. This is separate from the project's main testing-mindset rule file (step 13 reads that one, never writes it -- it is expected to already exist and auto-load).

## 15. Final summary and env files (last manual step)

Skill ends here -- don't pause for confirmation, don't verify the files, don't run the smoke tests. Print one gitignored block per env from 1.1, plus a committed `.env.test.example`. Each block has `BASE_URL=<env baseURL>` and one blank `<ROLE>_EMAIL`/`<ROLE>_PASSWORD` pair per authenticated role.

Use the baseURL the user gave for that env in 1.1 -- never a default host or port guessed here. If 1.1 produced no baseURL for an env, read the project's own `README.md`, `CLAUDE.md`, or `package.json` start script for it, and leave it blank with a `# TODO` rather than inventing one.

Render concretely with the project's real env and role names, e.g. for envs `local`+`qa`, roles `user`+`admin`:

> **Framework ready** -- last step, create the env files below.
>
> `.env.test.local` (gitignored):
>
> ```
> BASE_URL=<local baseURL from 1.1>
> USER_EMAIL=
> USER_PASSWORD=
> ADMIN_EMAIL=
> ADMIN_PASSWORD=
> ```
>
> `.env.test.qa` (gitignored): same shape with the qa `BASE_URL`.
>
> `.env.test.example` (commit): same shape, using the first env's `BASE_URL` as a placeholder.
