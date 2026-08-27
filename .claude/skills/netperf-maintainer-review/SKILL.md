---
name: netperf-maintainer-review
description: Review a redis-performance/netperf pull request, branch, or diff. This fork has NO independent review history whatsoever to ground a "maintainer voice" in — zero PRs ever opened against it, issues disabled repo-wide, no fork-specific AGENTS.md/CONTRIBUTING.md/README, and all 700+ commits authored upstream by HewlettPackard/netperf with none from redis-performance itself. Use this for any redis-performance/netperf PR review request instead of inventing maintainer precedent that doesn't exist — it applies generic, defensible C/autotools review judgment and says so plainly.
---

# netperf maintainer-style review

## Honesty note — read this first

This skill is adapted from an equivalent one built for `redis/memtier_benchmark` (~4 years of real, dialogic
maintainer review comments) and from `redis-performance/go-ycsb`'s (27 PRs, all from one author, five silent
`APPROVED` reviews from a second person). **`redis-performance/netperf` has neither.** As mined on 2026-08-27:

- **Zero pull requests, ever.** `gh pr list --repo redis-performance/netperf --state all --limit 100` returns
  nothing. No PR has been opened, merged, or closed against this fork at any point in its history.
- **Issues are disabled repository-wide.** `gh issue list --repo redis-performance/netperf` fails with "this
  repository has disabled issues." There has never been an issue thread here and none can be opened.
- **No fork-specific documentation exists.** No `AGENTS.md`, `CONTRIBUTING.md`, or even a redis-performance-authored
  `README` — the root `README` is HPE's original upstream text ("This is a brief readme file for the netperf
  TCP/UDP/sockets/etc performance benchmark..."), untouched.
- **No CI of any kind exists yet.** No `.github/workflows/`, no `.travis.yml`, no CircleCI config — nothing.
  This PR is the first CI this repository has ever had. There is currently zero automated build or test
  signal on any change to this repo.
- **The fork has not diverged from upstream at all.** `gh api .../compare/HewlettPackard:master...redis-performance:master`
  shows 0 commits ahead, 1 behind. Every one of the ~700 commits in this repo's history was authored by
  HewlettPackard/netperf's own maintainers (e.g. Gavin Brebner). redis-performance has never committed to this
  repo directly — it is a pure, unmodified passthrough fork, likely kept only to pin/mirror a known-good
  netperf revision for benchmarking use elsewhere in the org.

There is no maintainer voice to imitate, no nitpick taxonomy to mine, no policy document to cite, and no prior
review of any kind to pattern-match against. Do not fabricate any of these. Do not write a review as though
"this project's real reviewers" have a documented style — say plainly, if asked, that this fork has no
recorded review history, and fall back to generic, well-reasoned C/build-system review judgment instead. See
`references/grounding-note.md` for the full mining record (commands run, exact results) and
`references/review-checklist.md` for the generic checklist this skill actually applies, grounded in what the
codebase itself (not any review history) demonstrably is: an autotools-based C project with platform-conditional
network I/O code and no test suite.

## Process

1. **Get the material.** `gh pr view <n> --repo redis-performance/netperf --json body,commits,files,author` and
   `gh pr diff <n> --repo redis-performance/netperf`. The PR's own code is intentionally not checked out into
   the workspace — read it as text, don't execute it.

2. **Do not claim precedent that isn't there.** If the PR description or a commenter asks "would this pass
   review here" or "what would a maintainer say," answer honestly: this fork has no maintainers who have ever
   reviewed a PR, because none has ever been opened. Ground any opinion in the checklist in
   `references/review-checklist.md` and say so explicitly, rather than writing as though a real reviewer's
   habits are being channeled.

3. **Work the checklist** in `references/review-checklist.md`. It covers: build-system correctness (autotools —
   `configure.ac`/`Makefile.am`/`m4/` changes need a plausible story for `autogen.sh`/`configure` regeneration),
   platform-conditional code (this codebase has many `#ifdef`-gated, OS-specific source files — a change to
   shared code needs to not silently break the platforms it can't be tested against in one PR run), basic C
   memory/buffer safety, and CLI/flag-parsing changes (netperf's argument surface is large and undocumented
   beyond the `netperf.1`/manual — a new flag should be discoverable via `-h`/usage text, not silent).

4. **Because there is no CI at all**, do not defer any correctness judgment to "CI will catch it" — nothing
   will. Say plainly in the review if a change looks compile-risky or behaviorally risky and CI provides no
   safety net for it yet.

5. **Write the review terse, in plain prose, and as questions where appropriate** — there's no real precedent
   for tone here, so default to something closer to a careful outside reviewer's questions than a confident
   maintainer's verdict. If the PR is small, well-scoped, and self-evidently correct (a typo fix, a comment,
   a version bump), the honest output is often no comment at all (`skip_comment: true`) — don't manufacture
   nitpicks to look thorough on a fork with nothing to compare the PR against.

6. **No literal "Verdict:" label, no bolded summary line, no `@`-mention of any GitHub username** — these rules
   apply regardless of tone; see the workflow's own critical safety rules for why.

## What NOT to do

- Don't claim a "maintainer voice," a documented standard, or precedent of any kind for this fork — there is
  none. See the honesty note above.
- Don't invent or imply an `AGENTS.md`/`CONTRIBUTING.md` policy that doesn't exist in this repo.
- Don't defer to "CI will catch it" — there is no CI here besides the one this PR is adding, and it does not
  build or test the C code at all.
- Don't manufacture a duplicate-approval ("LGTM") comment on a routine PR just to produce output — silence is
  a defensible default when there's nothing substantive to say.
- Don't literally `@`-mention any GitHub username, ever, for any reason.
