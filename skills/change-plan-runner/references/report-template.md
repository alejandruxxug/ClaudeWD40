# Change report — <plan title>

- **Date:** YYYY-MM-DD
- **Repo / branch:** `<repo>` · `<type>/<plan-slug>` (from `origin/<integration branch>` @ `<hash>`), **not pushed**
- **Commits:** N (one per change)
- **Backend deploys:** none | dev: `<names>` (prod untouched) · **Frontend deploys:** none
- **Local build:** not run | ✅ | ❌ (see below)
- **Preflight baseline:** tests already failing before the run, console errors already present

## Summary

| # | Change | Commit | Tested on localhost | Status |
|---|--------|--------|---------------------|--------|
| 1 | … | `abc1234` | ✅ / ⚠️ partial (harness only) / ❌ | done / blocked / skipped |

## For you to verify after deploy

Items localhost couldn't prove. Each one: **where** (URL/screen) → **steps** →
**expected result** → **why not testable locally** → **what the harness covered**.

## Decisions made

Answers from the upfront questions, plus defaults chosen without asking (and why).

## Changes

### 1. <change title> — `abc1234`

**Request:** what was asked, in one line.

**Implementation:** what changed and why this approach. Key files as
`path:line` links.

**Tests run:**
- Happy path: …
- Edge cases: … (input → result)
- Permissions / layout / regressions: …
- Harness: which suites, pass/fail
- Console/network: clean vs baseline | …

**Issues found and fixed:** problem → cause → fix (each re-tested).

**Known limitations / not verified:** …

<!-- repeat per change -->

## Findings outside scope

Things noticed but deliberately not changed, with a suggested next step.

## How to review

- `git log --oneline origin/<integration branch>..<branch>`
- Drop a change: `git revert <hash>` (or rebase it out locally)
- Nothing pushed or deployed; push / PR when approved.
