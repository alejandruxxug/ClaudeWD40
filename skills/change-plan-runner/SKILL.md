---
name: change-plan-runner
description: Run a list of planned changes end to end, unattended, in any web project. Ask every clarifying question up front (each option explained with an example of what happens next), set up and prove a test environment (harness, dev server, connected Chrome, logins) before the user leaves, then for each change implement → test on localhost in Chrome (verify output, hunt edge cases and breaking points) → fix → retest → one local commit, narrating along the way. Finish with a short summary plus a full .md report, including anything the user must verify themselves. Use this whenever the user sends a list or plan of several changes/fixes/features to work through, says things like "here's the plan", "go through these changes", "leave it running", "test each one in chrome and commit separately", or asks for a change-by-change report — even if they don't name the skill.
---

# Change plan runner

The user hands over a list of changes and wants to walk away while you work through
it. Everything here serves one goal: when they come back, each change is done,
proven in the running app, isolated in its own commit, and explained well enough
that they can decide what to keep without re-deriving your work.

## Phase 0 — Learn the project, then ask everything at once

### Project profile
This skill works in any repo, so first learn this one's facts instead of assuming.
Read `CLAUDE.md` (and nested ones near the touched code), the memory index if there
is one, `package.json` scripts, and `git` remotes and branches. Note:

- **Dev server** command and URL (e.g. `npm start` → `localhost:3000`, `python manage.py runserver`, `bin/dev`).
- **Test harness** commands: unit tests, backend or emulator tests, linters, and
  what they need installed (e.g. an emulator, a local database, Docker).
- **Integration branch** to branch from (e.g. `dev`; fall back to the default
  branch) and the branch naming style.
- **Environments**: which one is dev and which is prod, and the commands that
  deploy to each. Anything that touches prod is off-limits for this skill.
- **Conventions**: language of code vs UI copy, commit style, how permissions or
  roles are gated, rules recorded in memory.

Anything the profile can't answer goes into the question round.

### Questions
The user leaves once questions are answered. A question you discover in change #4
means the run stalls or you guess, so front-load them.

1. Read the whole plan. For each item, read the code it touches (enough to know the
   files, the backend piece, the route, the permission involved).
2. Collect every ambiguity across all items and ask them in **one** round
   (`AskUserQuestion`, up to 4 questions per call; use several calls back to back if
   needed — still before starting work). Good questions are ones whose answer
   changes what you build: expected behavior, who sees it, copy text, edge-case
   policy. Don't ask what the code or a sensible default answers — decide, and note
   the decision in the report.

   **How to phrase each question.** The user decides once and then leaves, so each
   option has to show what their choice will actually change. Keep the question
   itself short (one line, naming the change it belongs to). Explain in the options:
   - **What I'll do:** the concrete change in the code or UI.
   - **Example of what happens next:** a specific scenario showing the result, e.g.
     who sees what, what a user clicks and gets, or what happens to existing data.
     Use the app's real screen names, roles and labels (in the UI's language).
   - **Trade-off,** when there is one, in a few words.

   Keep each option to two or three sentences. Use the `preview` field for anything
   visual (a small ASCII mockup of the screen, a table row, a message text) so the
   user can compare options side by side. Put the option you recommend first and
   mark it "(Recommended)".

   Example (change 3: "add an export button to the orders list"):

   > **Question:** Change 3: what should "Export" include?
   > - **Current filtered view (Recommended):** I export exactly the rows the list
   >   shows after search and filters. *Example: a user filters "Overdue" and sees
   >   12 rows; the CSV has those 12.* Fast, and matches what's on screen.
   > - **Every record:** I ignore the filters and export the whole table on the
   >   backend. *Example: the same user gets all 3,400 rows and has to filter in a
   >   spreadsheet; the download takes several seconds.* Needs a new API endpoint
   >   and a dev deploy.

   Bad: "Should the export respect filters?" A yes/no with no consequences leaves
   the user guessing what each answer builds.
3. Always settle the run's logistics in that same round unless already known.
   Phrase these the same way, with the consequence of each answer:
   - **Test accounts** in Chrome (which user/role to log in as; several roles if
     permissions are involved).
   - **Backend changes**: may you deploy backend code (functions, APIs) to the
     **dev** environment to test it from localhost? *Example: "Yes": after change 2
     I deploy the changed function to dev and test the new filter in Chrome against
     it. "No": I cover it with the backend test harness only, and the report lists
     it under "for you to verify".*
   - **Writes to dev data** while testing (creating records, sending dev emails) — OK?
   - **Local build**, asked only when the plan touches build config, env vars,
     routing or dependencies. *Example: "Yes": after the last change I run the
     production build locally to prove the bundle compiles with every change;
     nothing leaves the machine. "No": the report says the bundle wasn't checked.*
4. Restate the plan as a numbered list with your interpretation of each item and the
   order you'll do them (dependencies first).

## Localhost first; no frontend deploys

All browser testing happens on the **local dev server**. Never deploy the frontend
(hosting, static site, preview channels) as part of this run: a deploy publishes
every commit on the branch, including ones the user may reject.

Some things genuinely can't be verified on localhost: e.g. captcha or bot protection
bound to the real domain, OAuth or email-link redirects, CORS or CSP that only
applies on the deployed origin, service workers, CDN headers. When a change hits
one of these:
1. Don't deploy, and don't stall. Drive and verify everything that *does* work
   locally, and cover the rest with the test harness (unit tests, backend or
   emulator tests, a test you write for the case).
2. Commit the change as normal.
3. Add it to the report's **"For you to verify after deploy"** section: what to
   open, what to do, what the correct result looks like, and why localhost couldn't
   show it. Mention it in the final chat summary too, because this part is the
   user's job.

## Phase 1 — Preflight, while the user is still here

The user leaves once you start. Anything that needs them later (a login, a browser
permission prompt, a missing tool) would stall the whole run, so set up and **prove**
the test environment now. Narrate each check, fix what you can yourself, and ask
the user only for what needs them (a password, an OTP, clicking "Allow" in Chrome).

| # | Check | How | If it fails |
|---|-------|-----|-------------|
| 1 | **Git** | `git status` clean; `git fetch origin`; `git checkout -b <type>/<plan-slug> origin/<integration branch>` | Uncommitted work: ask. Never stash or discard it on your own. |
| 2 | **Dependencies** | Packages installed and matching the lockfile (install if not); local env files the dev server needs exist | Missing env file: ask the user. Never invent credentials. |
| 3 | **Test harness baseline** | Run every relevant test command from the profile once, plus the tooling it needs (`--version` checks) | Record the tests that **already fail** before any change, so they aren't blamed on the run. Missing tooling: tell the user now. |
| 4 | **Dev server** | Start it in the background; wait until it compiles without errors and the URL responds | Port busy: reuse the server if it's this repo's, otherwise ask. |
| 5 | **Browser connected** | Prefer Claude in Chrome: load its tools in **one** ToolSearch call (tabs_context_mcp, tabs_create_mcp, navigate, computer, read_page, find, form_input, javascript_tool, read_console_messages, read_network_requests, tabs_close_mcp, list_connected_browsers); call `tabs_context_mcp`; open a fresh tab on the dev URL. If it isn't installed, use whatever browser automation the session has (built-in browser, Playwright MCP) and map the checks below onto it | Extension unreachable or several browsers: ask the user to connect or pick one. No browser tooling at all: say so now, because every change will then be harness-only. |
| 6 | **Site permission** | Do one real action on localhost (click, read the page, run a short `javascript_tool` snippet) | Permission prompt: the user approves it **now**, not mid-run. |
| 7 | **Logged in** | Log in as every test account the plan needs; confirm the start screen loads with that role's menus | OTP, password or captcha: the user enters it now. Note which tab or session holds which role. |
| 8 | **Console baseline** | `read_console_messages` with an error pattern on the start screen | Record existing errors and warnings, so later checks only flag new ones. |
| 9 | **Deploy access** (only if dev backend deploys were allowed) | The deploy CLI (gcloud, firebase, aws, vercel, …) points at the **dev** project and auth is valid | Re-auth needed: the user runs it now with `! <login command>`. |

Finish preflight with a short checklist in chat (✅/⚠️ per row, plus anything you
recorded as a baseline). Then hand off explicitly, e.g. *"Environment ready: branch
`feat/requests-export`, dev server on :3000, Chrome logged in as ADMIN and USER,
2 tests already failing (noted). Starting change 1/5. You can leave."* Don't wait
for another go-ahead after that.

From here on, run without stopping. If something truly blocks a change (a denied
permission, an expired session you can't renew), record it, move on to the next
change, and cover it in the report. Don't halt the whole run. If the dev server or
Chrome drops mid-run, re-run the matching preflight rows before continuing.

## Per change: change → test → fix → test again → commit

Narrate each step in one or two lines as you go ("Change 2/5: … — implementing",
"testing the empty state", "found: …, fixing", "retest passed, committing"). The
user may read the scrollback later; it should tell the story without the report.

### 1. Change
Implement the smallest change that fully does the job, matching the surrounding
code and the project's conventions. Run the affected tests from the harness.

### 2. Test in Chrome (localhost)
Exercise the real UI, not just the happy path. Write down (for the report) what you
checked. Cover, as applicable:
- **Happy path** — the change does what was asked; screenshot the result.
- **Edge inputs** — empty, very long text, special characters/accents/emoji, zero,
  negatives, max values, duplicates, dates at boundaries.
- **States** — loading, empty list, error from the backend, slow network, reload
  mid-flow, back button, opening the URL directly.
- **Permissions** — a user who should see it and one who shouldn't (UI hidden *and*
  backend rejects).
- **Layout** — narrow window (~400px) and wide; long values don't break tables.
- **Regressions** — the neighboring flows on the same screen still work.
- **Console & network** — `read_console_messages` (filter by error) and failed
  requests in `read_network_requests`, compared against the baseline.

Never trigger `alert`/`confirm` dialogs — they freeze the extension. If one is
unavoidable, say so in the report and leave it for the user.

### 3. Fix, then test again
Each problem found: fix it, then repeat the checks that found it **plus** the happy
path (fixes break things). Loop until clean. If an issue is out of scope or a
judgment call, don't silently expand scope — note it in the report as a finding.

If Chrome tooling fails 2–3 times in a row, stop poking the browser: fall back to
the harness and code review for that change and list it under "for you to verify".

### 4. Commit (local only)
One commit per change, in the project's commit style, body says *why*, with the
session's attribution trailer. Stage only that change's files. **Never push, never
open a PR, never deploy the frontend, never touch prod** — the user reviews first
and decides what survives. Record the short hash for the report.

## Finish

1. If a local build was allowed, run it now with every change in place. A failed
   build is a defect: fix it in a separate `fix` commit, then build again.
2. Stop the dev server and close the tabs you opened.
3. Write the full report using `references/report-template.md` to
   `docs/change-reports/YYYY-MM-DD-<plan-slug>.md` (or the repo's existing docs
   folder if it uses another layout). Leave it **uncommitted**.
4. Reply in chat with a short summary: one or two lines per change (what, how,
   commit hash, test verdict), then the **"for you to verify after deploy"** items,
   the open findings, and the report path. Keep it short — the report holds the
   detail.
5. Tell the user that nothing has been pushed or deployed, and how to drop a change
   they reject (`git revert <hash>`, or rebase it out locally).
