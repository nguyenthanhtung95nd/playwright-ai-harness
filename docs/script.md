# Playwright AI Harness -- Session Script

**Format:** ~130 minutes total, one track, QA/Dev/PM/BA in the same room.
**Timing:** ~58 min concept (sections 1-14) -> ~35 min live demo (section 15) -> ~22 min hands-on get-started (sections 16-23) -> ~10 min Q&A (unscripted) -> ~4 min close (section 24).
**How to use this script:** each section is written to be spoken, not read verbatim from slides -- the slide carries the evidence table, diagram, or code, the speaker carries the narrative connecting them. Bracketed notes `[like this]` are stage directions, not spoken text.
**Every claim in this deck points at a file in this repo.** If someone in the room asks "where does that come from," the answer is a path under `.claude/`, not a citation. Open the file.

---

## 1. The Cost Problem

_(~4 min -- opens on business terms, no tool names yet)_

Every team eventually hits the same wall: automated tests exist, but nobody fully trusts them. A failure shows up and the first reaction isn't "what broke" -- it's "is this real, or is this just flaky again." Once that question becomes routine, the test suite has stopped doing its job. People stop blocking releases on it. They re-run until it's green. The suite is still running, technically, but it has quietly been demoted from safety net to background noise.

This isn't a tooling problem -- Playwright, Cypress, Selenium, it doesn't matter which. It's a discipline problem: nobody wrote down, anywhere, _what a good test is allowed to do and what it isn't_. Without that, every contributor -- human or AI -- makes a locally reasonable choice (add a sleep, retry until green, assert less) that's globally corrosive.

That's the premise for today: the fix isn't a smarter tool, it's a **written, enforced harness** -- and once an AI agent is writing the tests, that harness stops being a nice-to-have onboarding doc and becomes the only thing standing between "the agent said it's done" and "it's actually done."

---

## 2. The Full-Stack Mindset

_(~4 min)_

[Slide: shift-left swimlane diagram]

Before getting into what a harness looks like, it's worth grounding _why_ any of this matters -- and the answer predates AI agents entirely. Testing was traditionally a siloed activity: build it, then throw it over the wall for someone else to test. Full stack testing flips that -- quality checks happen in parallel with development, not after it. That's shift-left testing: a three-amigos conversation and a design review happen _before_ a line of code is written. Unit tests, CI runs, and dev-box exploratory checks happen _while_ it's being written.

The payoff is concrete: before shifting left, a team gets all its feedback after development is "done." After, roughly half of that feedback arrives before the story even reaches a formal test phase.

And here's the sentence that frames everything else in this talk: **quality is the team's responsibility.** Not QA's, not one engineer's -- everyone's. If that's true, then the rules for what makes a good test have to be written down and portable to everyone on the team, not kept in one person's head. That's the premise this whole harness rests on.

---

## 3. What a Harness Is

_(~5 min)_

[Slide: two-card definition + two callouts]

Strip away the branding and an AI harness is two things, nothing more:

1. **One mindset document** -- the handful of non-negotiable rules a reviewer would enforce by hand anyway (how tests get their data, how they wait for async state, where assertions are allowed to live).
2. **A small number of lifecycle playbooks** -- plan, build-and-verify, ship -- each one a repeatable sequence the agent follows, not a one-off instruction typed fresh every time.

The reason this matters more with AI than it did with humans is **context engineering**: an agent only behaves as well as the information in front of it _at the moment it decides something_. A human engineer carries two years of "we don't do that here" in their head. An agent carries nothing between sessions unless you hand it back explicitly. If the rule isn't written down and loaded, it doesn't exist for that run.

The second half is what makes this trustworthy: an **external reinforcement loop**. Left alone, an agent's incentive is to report success -- it will call a test "done" the moment it stops erroring, not the moment it's actually correct. A test suite, a rules-aware audit, a git hook that blocks a bad commit -- these are loops _outside_ the agent's own judgment that it cannot talk its way past.

**Land this line:** "the harness is not the rules file by itself. It's the rules file plus something automatic that checks whether the rules were followed."

---

## 4. Rules vs Skills

_(~4 min)_

[Slide: two-card comparison -- rules / skills, plus the permissions callout]

Part one and part two don't just differ in content -- they differ in how an agent even gets access to them. A **rule** file loads into every single conversation automatically, whether this turn needs it or not. That's why the mindset document has to stay short: nothing can start a test without it already loaded. A **skill** is different -- it's discovered by its own description when the task actually matches, or invoked by name on purpose. The lifecycle playbooks live here, so a conversation that isn't planning or shipping a test never has to carry their full detail.

Worth saying plainly: a skill is **not a prompt template**. It bundles domain knowledge with a repeatable workflow, which is exactly why `pw_plan`, `pw_work`, and `pw_setup` are multi-step documents with named modes rather than one-line instructions.

**Narration point:** "and there's a third layer below both of those, which people miss. `.claude/settings.json` in this repo denies `git commit` and `git push` outright. A rule can be reasoned around. A denied permission cannot. That's the cheapest external reinforcement loop there is -- open the file, it's eight lines."

Not everything an agent might need belongs in every conversation. Rules are the floor that's never optional. Skills are the toolbox reached for only when the job calls for it. Permissions are the wall neither one can go through.

---

## 5. The Test Pyramid

_(~4 min)_

[Slide: three-tier pyramid diagram]

The shape of a good test suite isn't a matter of taste. One line of strategy: test the right scope of functionality at the layer that gives the team the fastest feedback. As scope grows -- from a unit test up to a full UI-driven test -- each one costs more to write, run, and maintain. So the suite should taper: a broad base of unit and integration tests, a narrower band of service tests, and a small, expensive apex of UI-driven tests.

Make the audience run the arithmetic on their own suite, don't hand them someone else's number: a unit test finishes in milliseconds. A UI-driven test that logs in, navigates, and waits on real network round trips takes 30 to 90 seconds. Two hundred of those is a nightly job, not a feedback loop. The same coverage proved one layer down finishes before the coffee does.

One honest caveat: the ideal pyramid isn't always reachable -- missing test environments, a tooling gap, a missing skill. When that happens, the move is to be deliberate about the trade-off, not silent about it.

---

## 6. Antipatterns

_(~4 min)_

[Slide: ice-cream cone vs. cupcake comparison]

Two ways this shape breaks, and an agent that can generate a dozen tests in a minute makes both worse. Invert the pyramid -- many UI tests, almost no unit tests -- and you get the **ice-cream cone**: long waits for feedback, defects caught only at release, and a team that still does manual testing despite having "automation." The earliest tell is finding regression defects during manual story testing -- root-cause that immediately, before it compounds.

Duplicate the pyramid instead -- the same case tested again at every layer -- and you get the **cupcake**. It usually shows up when developers and testers work in silos, each adding their own version of the same case. AI-assisted bulk generation makes this worse, because it's fast to generate coverage without checking what already exists.

This is exactly why "could a lower layer prove this cheaper" is Rule 1 in `testing-mindset.md` -- not a style preference, a defense against both of these failure modes.

---

## 7. Autonomy Has a Floor

_(~3 min)_

[Slide: the verbatim line from all three SKILL.md files + two cards]

All three skills in this harness open with the same line. Not paraphrased -- the same string, in all three files, and it's on the slide exactly as it appears on disk:

> **Never skip `AskUserQuestion` steps in this skill, even if told to work autonomously.**

That's deliberate. The checkpoint isn't left to the agent's judgment about whether pausing is worth it -- it's the first instruction it reads, before any mode is chosen. Two places where that floor actually bites:

- `pw_plan` enters **plan mode** before anything else. It reads the existing specs and the app source, brainstorms cases, and writes no file at all until a human approves the plan.
- `pw_work`'s **SHIP** mode stages the change and drafts a message. It never runs `git commit` or `git push`, in any mode, _even if the user says yes to a commit prompt_. That hard rule is in `pw_work/SKILL.md`, and the permission deny-list behind it means it couldn't anyway.

**Narration point:** "an agent told to 'just get it done' still can't dissolve the checkpoint, because the checkpoint isn't a suggestion it's weighing. It's an instruction it already read."

---

## 8. Debugging Discipline

_(~4 min)_

[Slide: three evidence-source cards]

`pw_work`'s VERIFY mode carries three debug branches, split only by where the evidence comes from: a saved trace for a local failure, a live attach when a static trace can't explain a race condition, and a downloaded CI artifact when it only fails in the pipeline. Three branches for three evidence sources -- but all three converge on the exact same first move before touching any code.

Before fixing anything: **name which bucket the failure is actually in.** Test bug, application bug, environment or config problem, or flake. That line is in `VERIFY.md` near the top, and it says the bucket you picked and why is _the most important part of the report_ -- more important than the fix.

And across all three branches, the same hard line: a flake or an application bug doesn't get fixed with a longer timeout, a retry, or `force: true`. That just hides it from the next person who hits it.

[Section 20 shows the actual commands. Here it's only the discipline.]

---

## 9. Audit Discipline

_(~3 min)_

[Slide: two-card comparison -- scope / posture, plus the success-criteria callout]

VERIFY's audit branch runs deliberately as a fresh session, with no memory of writing the code it's reviewing. Scope is strict: `git status --short` and `git diff HEAD` define the **entire** audit scope. No uncommitted changes means nothing to review -- stop. A test that's already committed doesn't get silently re-opened.

The posture matters as much as the scope: the reviewing agent is explicitly told it is _not_ the agent that wrote the code, and to question every choice. Every finding has to cite the specific rule it violates -- an opinion with no rule behind it doesn't belong in the report. And the report isn't just a wall of red: findings split into must-fix, should-fix, and suggestions, plus a section for what was done right.

**Narration point:** "autonomous fixing only works when 'done' was defined before the agent started. Here it's exact, and it's the same command every time -- `npx playwright test -g "<title>" --retries 0` passes, and nothing else broke. There's nothing left for the agent to interpret."

---

## 10. Personal vs Project

_(~3 min)_

[Slide: two-card comparison -- on your machine / for everyone else]

This slide exists because of one specific fact about how Claude Code resolves a skill name: if the same skill exists both in your personal setup and in a project, **your personal copy wins, always**, on your own machine. Pushing a copy into the project doesn't change that for you.

So why does `pw_setup`'s SHARE mode exist at all? For everyone who isn't you: teammates without your personal setup, and CI machines that have no personal copy to fall back on. That's the entire reason to commit a project copy.

And when the two drift apart -- your personal copy gets updated, the project's doesn't, or vice versa -- SYNC does not auto-merge. For every item that differs, a human picks the winning side, one at a time.

**Narration point:** "same judgment-over-automation stance as everything else today -- a human decides, the tool never guesses which version is right."

---

## 11. Pillar One -- Judgment Over Compliance

_(~7 min)_

[Slide: 4-row comparison table]

Four places where this harness is deliberately stricter than the default advice most people in this room have absorbed. Each one trades convenience for a specific defect class it refuses to let through. The point isn't that the common advice is wrong -- it's that a default is a floor, and these four are worth raising on purpose.

| Common guidance                                                                                | What we actually enforce                                                                                | Why we went further                                                                                                                                                                                                           |
| ---------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Retries are an acceptable bandage for "genuinely flaky" transient failures -- use them judiciously. | Retries are disabled entirely (`retries: 0`). A flake is treated as a defect report, not noise to smooth over. | A flaky test that passes on retry is a race condition wearing a disguise. Allowing _any_ retry gives every future contributor a legitimate-looking place to hide the next one. Zero tolerance makes hiding impossible, not just discouraged. |
| Database-backed data pools are a legitimate strategy for managing expensive or finite test data. | No direct database writes from tests, ever. All test data is created through the same real user flows a human would use. | A pool only proves the database accepts rows -- it says nothing about whether the application's own flow for creating that data still works. The app's write path breaking silently is exactly the defect class this is designed to catch. |
| Stable locators should prioritize dedicated test-id attributes above semantic/role-based locators. | Role- and label-based locators only -- a missing or ambiguous accessible name is treated as a defect to fix, never worked around. | Picking by role/label only passes when the name, role, and state are actually correct for assistive technology. The locator strategy itself becomes a free, continuous accessibility check -- a test-id attribute would hide exactly the defect this is meant to surface. |
| AI-assisted self-healing locators are a meaningful reliability improvement -- let the tool adapt when the page changes. | Locator ambiguity is flagged for a human to fix, never auto-healed. | An auto-healed locator can silently start pointing at the wrong element after a redesign and still pass. Self-healing optimizes for the test staying green. This harness optimizes for the test staying _honest_ -- a stricter, and deliberately less convenient, bar. |

The pattern across all four: in each case, the "reasonable default" optimizes for the test suite looking healthy. The stricter rule optimizes for the test suite actually catching something real -- a conscious trade of short-term convenience for long-term trust.

---

## 12. Network Determinism

_(~4 min)_

[Slide: three code blocks -- record / replay / capture from a browser]

The discipline just argued for locators and test data applies one layer down, to the network itself. This one is codified in `.claude/rules/playwright-scripting.md`, so it's a rule, not a tip.

The app's real network traffic gets captured to an HTTP archive during an actual run -- `routeFromHAR(..., { update: true })` writes it. Then you commit that archive and flip to `update: false`, and every run after replays the recorded traffic instead of hitting a live, possibly flaky or rate-limited API. Same response, every time, whether the real service happens to be up or not. You can also capture the archive straight from a browser session with `npx playwright open --save-har=...` and no test at all.

**Narration point:** "this isn't mocking. A recorded archive is something that actually happened; a hand-written mock is a guess that rots silently when the real contract changes."

Two warnings worth saying out loud, because both are mistakes people make on day one:

- **Scope the `url` option** to the endpoints the test actually depends on. A HAR that captures fonts, analytics, and images is large, slow to replay, and breaks on changes with nothing to do with the test.
- **Route interception is the exception, not the default.** Use it for a state a real run can't produce -- a 500, a timeout, an empty list. Never intercept the app's own create-path to fake test data: that removes the one check Rule 4 exists to protect.

---

## 13. Pillar Two -- Lean Beats Sprawling

_(~5 min)_

[Slide: the full harness as a tree -- 3 skills, 7 modes, 10 branches]

If judgment is the first pillar, scope discipline is the second -- and this is the one engineers in the room will feel in their day-to-day fastest.

The obvious way to add capability to a harness is to add another skill file. One to debug a test that failed locally, a second for one that failed live, a third for CI. One to generate a test when source is available, another when it isn't. Each one is individually reasonable. Collectively they're a problem: every file is a new thing a person has to remember exists and choose correctly between, and they drift out of sync with each other -- which usually gets "fixed" by adding yet another file whose only job is reconciling the rest.

The slide is the entire harness, on one screen: **three skills, seven modes, ten branches.** `pw_plan` does one thing. `pw_work` has three modes and six branches inside them. `pw_setup` has three. Every bit of that depth is still there -- it just lives inside a workflow instead of at the top level.

And the choice a person has to make got easier, not harder: you pick by _where you are in the work_ -- planning, building, setting up -- which you already know without anyone telling you.

**The lesson generalizes past Playwright:** count the things a person must choose between, not the capabilities. Depth inside a mode is free. A new file at the top level is not. Skill count is a cost, not a feature.

---

## 14. From Ticket to Green Test

_(~3 min)_

[Slide: five-stage pipeline diagram -- pw_plan -> IMPLEMENT -> VERIFY -> SHIP -> human commits]

Pillar 2 made the case in the abstract. Here it is as a pipeline on one ticket: `pw_plan` turns the ticket into a reviewed plan. `pw_work`'s IMPLEMENT writes the test. VERIFY audits it with fresh eyes or debugs it if it's red. SHIP stages it.

The SHIP stage is the one gate worth calling out by name: it stages the change and drafts a commit message, but it never runs `git commit` or `git push` itself -- even if asked to. A human takes it from there.

**Narration point:** "four stages, two skills covering them, one place where a human has to act. The step count and the skill count are different numbers, and that's the whole point of the last slide."

---

## 15. Live Demo

_(~35 min -- narration script. Actual commands/screens are the presenter's, not scripted here)_

[Switch to the demo machine. App: Conduit, a public Angular blogging app. Nobody in the room has worked in this codebase.]

**Framing line before starting:** "Everything from here is happening on an app none of us have touched. If this only worked on my own project, it wouldn't prove anything -- so we're doing it live, somewhere neither of us has an advantage."

### Checkpoint A -- Init rules

Run `pw_setup` INIT from a blank slate and narrate each question as it comes: how many environments, which auth roles, how to find the login flow, whether to wire the audit into a pre-commit hook. Then walk the mindset document it installs, rule by rule.

**Narration point:** "Notice how short the rule file is. If it needs to be longer than a page or two, that's usually a sign the rules are trying to cover something the architecture should be handling instead."

### Checkpoint B -- Coverage plan

Before any test code exists, run `pw_plan` and produce a reviewable plan of what will be tested and why -- specifically calling out anything that _shouldn't_ be an end-to-end test because a lower-level test could prove the same thing cheaper.

**Narration point:** "This step is what stops the harness from generating busywork. A plan you can read and reject in two minutes is worth more than a test suite you have to discover is wrong six months later."

### Checkpoint C -- Audit

Introduce a test that was deliberately written to break a rule from Checkpoint A (a fixed wait, or an assertion placed in a page object) and run `pw_work` VERIFY against it. Show that every finding cites the rule it violates.

**Narration point:** "This is the part that matters most: the rule only means something if it's checked by something other than good intentions."

_(If time remains:)_ attempt the commit and let the pre-commit hook reject it.

**Narration point:** "Now it's not a review step someone can skip under deadline pressure -- it's a gate."

---

## 16. Install the Rule and the Skills

_(~3 min)_

[Slide: four-item reference + the `/pw_setup` command block]

Not an abstraction -- real files that already exist in this repo. `testing-mindset.md`: eight rules, each with a stated why, the same file the audit from Checkpoint C just checked against. And `pw_setup`, with three jobs: **INIT** scaffolds a brand-new Playwright framework and writes the rules in one pass. **SHARE** copies `pw_plan`, `pw_work`, and `pw_setup` itself from your personal setup into a project, so teammates and CI get the same workflow. **SYNC** reconciles drift between the two copies, your call on which side wins per item.

Walk the command block on the slide: it asks before it writes anything -- environments and their baseURLs, auth roles, how to find the login flow, whether to wire the hook. Then it writes `e2e/`, `playwright.config.ts`, `playwright-utils/`, the rules, and `.husky/pre-commit`.

**Narration point:** "Run `pw_setup` once per project. That installs the rule and shares the skill set in the same pass -- there's nothing else to set up."

---

## 17. A Real Ticket, End to End

_(~5 min -- the flagship worked example. Slow down here.)_

[Slide: the full command transcript]

This is the whole loop typed out, nothing paraphrased. Walk the slide top to bottom, out loud:

A ticket arrives -- a signed-in user can favorite an article from the feed, and the count goes up.

`/pw_plan a signed-in user favorites an article from the global feed` -- it enters plan mode, which is read-only. It reads the existing specs, the component source, and the rule file. It comes back with P0/P1/P2 cases _and_ what shouldn't be an E2E test at all. You approve, and only then does a file get written, to `.claude/pw-plans/`.

`/pw_work implement the first P0 case from favorite-article-from-feed` -- it reads the plan, the existing specs, the component source, writes the spec, then asks before running it.

`/pw_work verify` -- fresh-eyes audit, scoped to `git diff HEAD` and nothing else. Must-fix, should-fix, suggestions, what looks good. Every finding cites a rule.

`/pw_work ship` -- stages the change, drafts the message, stops. You run `git commit`.

**Narration point:** "and you don't have to name the mode. 'Debug this failing test' goes straight to VERIFY's debug branch. 'Refactor these into page objects' goes to IMPLEMENT's branch B. The skill states which mode it picked before it acts, so if it guessed wrong, you correct it in one sentence."

---

## 18. Auth, Solved Once

_(~3 min)_

[Slide: `auth.setup.ts` and the matching `playwright.config.ts` projects block, side by side]

One thing worth calling out explicitly before anyone opens a terminal: `pw_setup` already solves authentication. Logging in through the real UI on every single test multiplies run time, and multiplies the chances that the login flow itself breaks the thing a test wasn't even trying to check.

INIT writes an `auth.setup.ts` that logs in once per role and saves `storageState` to disk. In config, that becomes its own project, and every role-specific project declares `dependencies: ['auth-setup']` plus the storage file it should load. Swapping a test to a different role is a config change, not a rewrite. Guest projects simply omit `storageState` -- unauthenticated on purpose.

**Narration point:** "look at what isn't in the test file: no credentials, no URL. Both come from `.env.test.<env>`, which is gitignored, and every test navigates with relative paths so `baseURL` stays the one place an environment is ever named. A hardcoded host in a test is how a suite ends up only running on one person's machine."

---

## 19. Codegen and the CLI

_(~3 min)_

[Slide: codegen commands + playwright-cli commands]

Rule 2 says never guess a locator. These are the two tools that mean you never have to.

**Codegen**, when the app is in front of you: `npx playwright codegen "$BASE_URL"` opens a browser, records every click and fill as you perform the real flow once, and writes working test code as it goes. The `--load-storage` flag starts you from an already-signed-in session instead of making you log in by hand every time.

**The CLI**, when the source isn't yours to read -- a third-party site, another team's app, a deployed-only environment: `playwright-cli open`, then `snapshot` to see roles and names, `click` to walk the flow, and `generate-locator <ref>` to _ask_ for the locator rather than guess it. This is the path `pw_work`'s IMPLEMENT mode takes when it can't read source.

**Narration point:** "treat both outputs as drafts, not tests. Codegen will happily emit a CSS chain. IMPLEMENT re-checks every locator against the priority order -- role, label, text, placeholder, testid, and only then CSS -- and replaces what doesn't hold up. And when no semantic locator exists at all: if the source is yours, the fix is a `data-testid` added to the component, not a class selector. If it isn't yours, the skill flags it so you can ask the owning team for one."

---

## 20. When a Test Goes Red

_(~4 min)_

[Slide: the full debug command transcript]

Section 8 gave the discipline. This is what it looks like in a terminal.

`/pw_work debug "<test title>"`, and then three steps. **One:** re-run the test with tracing forced on and retries off. **Two:** read the trace from the CLI -- `npx playwright trace`, then `open`, `actions --grep="expect"` to find the step marked with an X, `action <n>` to see what it tried and what it got, `snapshot <n> --name after` for the DOM at that instant, `requests` to check whether the API call even fired, `console` for app-side errors. Not the GUI viewer -- that blocks the agent, which is exactly why the skill says to use the CLI.

**Three, and this is the one that matters:** before touching any code, say which bucket it's in. Test bug, application bug, environment or config problem, or flake.

**Narration point:** "two other branches exist for the same job. When a static trace can't explain a race, VERIFY pauses the live run at an exact line and probes the real page -- without editing your spec to insert a `page.pause()`. When it only fails in CI, it pulls the trace out of the failed workflow run with `gh` and treats that as authoritative, rather than a local re-run that may not reproduce it at all."

**And the line to leave them with:** "a longer timeout, a retry, or `force: true` is never a fix for an application bug or a flake. It just hides it from whoever hits it next -- and that person is usually you, in three weeks, with less context."

---

## 21. Make the Rule a Gate

_(~3 min)_

[Slide: `audit.mjs` CHECKS array and `.husky/pre-commit`, side by side]

Rule 8 is the one that makes the other seven real: a rule only checked by memory is not a rule, it's a hope. Anything checkable mechanically gets checked mechanically.

`pw_setup` INIT wires this up, and it asks you which rules to include rather than deciding for you. Each entry in `CHECKS` is a rule name, a pattern, and the message a developer sees. The hook runs `lint-staged` and then `node e2e/audit.mjs`; a non-zero exit blocks the commit. The output cites the rule, the file, the line, and the offending code -- never an opinion.

**Narration point, and say this one plainly:** "two of the eight rules have a mechanical check today. 'One behavior per test' and 'could a lower layer prove this cheaper' have no reliable pattern, so they stay review-only. That gap is the backlog, not a claim that the other six matter less -- and the rule file documents that about itself on purpose, rather than pretending it's all covered. A harness that overstates its own coverage is worse than one that admits the gap."

---

## 22. Write Your First Test Right Now

_(~3 min -- the hands-on stretch. Give the room a minute of quiet to actually do this)_

[Slide: three-step checkpoint row -- pw_plan / pw_work implement+verify / pw_work ship]

**Framing line:** "Pick one real scenario from your own app -- not a toy example, something your team actually needs covered. You have everything you need to start before you leave this room."

Three skill calls, start to finish. `pw_plan`: describe the scenario, it researches the real code and won't write a file until the plan is approved. `pw_work`'s IMPLEMENT and VERIFY modes: the test gets written and run, then a fresh pass audits it against `testing-mindset.md`'s own rules and reads the failure evidence if it's red. `pw_work`'s SHIP mode: drafts a summary and stages the change -- you still press commit yourself.

_(If the room's energy supports it, actually pause here and let people open their own terminal.)_

---

## 23. Before You Leave This Room

_(~2 min)_

[Slide: three-item checklist]

Quick, concrete close to the hands-on stretch: `pw_setup` has run once for this project, not just this demo repo. One real coverage plan exists from `pw_plan`, for one real scenario, reviewed before any test code was written. And `pw_work` has run at least once, even just the audit branch, so you've seen what it finds.

**Narration point:** "If you only do one of these three today, make it the plan -- it's the cheapest proof that this works on your codebase, not just this demo app."

---

## 24. Closing -- Anyone Can Reach It Now

_(~4 min)_

Step back from the demo for a moment. Nothing in the last stretch required deep Playwright expertise from the person running it -- it required a written mindset, a plan reviewed before code, and a check that enforces itself. That's the actual point of building a harness this way: it moves the hard-won judgment of an experienced test engineer _out of one person's head_ and _into a document the whole team -- and every agent working for the team -- can load and be held to_.

That's what turns "QA owns quality" into something closer to the truth: **when the discipline is written down and enforced automatically, any engineer who can describe a user journey can own the test quality for their own feature.** Not because the bar got lower -- because now anyone can reach it.

If there's one thing to take out of this room today, it's not a tool. It's the one question to ask before adding anything to your own team's harness: _is this rule actually enforced, or just hoped for?_
