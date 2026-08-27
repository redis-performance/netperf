# Grounding note: what was actually mined from redis-performance/netperf (2026-08-27)

This file records the exact commands run and their exact results, so anyone auditing this skill later can
verify the honesty note in `SKILL.md` rather than take it on faith.

## PR history

```
$ unset GH_TOKEN; gh pr list --repo redis-performance/netperf --state all --limit 100
```
Output: empty. Zero PRs — open, closed, or merged — exist against this fork.

## Issue history

```
$ unset GH_TOKEN; gh issue list --repo redis-performance/netperf --state all --limit 100
```
Output: `the 'redis-performance/netperf' repository has disabled issues`. Issues are turned off at the repo
settings level; there is no issue history to have mined even if one had wanted to.

## Repo metadata

```
$ unset GH_TOKEN; gh repo view redis-performance/netperf --json defaultBranchRef,description,isFork,parent
```
Result: `isFork: true`, `parent: HewlettPackard/netperf`, default branch `master`.

## Divergence from upstream

```
$ unset GH_TOKEN; gh api repos/redis-performance/netperf/compare/HewlettPackard:master...redis-performance:master \
    --jq '.ahead_by, .behind_by, .total_commits'
```
Result: `0` ahead, `1` behind, `0` total commits unique to this fork. redis-performance's `master` has never
had a single commit made against it that isn't already in the upstream HPE repository — it is actually
slightly *behind* upstream at time of writing.

## Commit authorship

```
$ unset GH_TOKEN; gh api repos/redis-performance/netperf/commits --paginate -q '.[].sha' | wc -l
```
Result: 703 commits total, all reachable from `master`. Sampling the most recent ones shows authors like
"Gavin Brebner" with messages such as "Merge pull request #54 from GavinB-hpe/UpdateLicenses" — i.e. upstream
HPE maintainer activity, not redis-performance activity. No commit found in a spot-check was attributable to a
redis-performance-affiliated author.

## Fork-specific documentation

```
$ unset GH_TOKEN; gh api repos/redis-performance/netperf/git/trees/master?recursive=true -q '.tree[].path' \
    | grep -iE 'agents|contributing|readme'
```
Result: only the upstream `README` and its platform variants (`README.aix`, `README.hpux`, `README.osx`,
`README.ovms`, `README.solaris`, `README.vmware`, `README.windows`). No `AGENTS.md`, no `CONTRIBUTING.md`. The
root `README` content itself is unmodified HPE boilerplate ("BE SURE TO READ THE MANUAL...").

## Existing CI

```
$ unset GH_TOKEN; gh api repos/redis-performance/netperf/git/trees/master?recursive=true -q '.tree[].path' \
    | grep -iE '\.yml$|\.yaml$|travis|circleci|jenkins'
```
Result: empty. No `.github/workflows/`, no Travis, no CircleCI, no Jenkinsfile. This PR introduces the first
CI configuration this repository has ever had.

## Language / project shape

```
$ unset GH_TOKEN; gh api repos/redis-performance/netperf/languages
```
Result: `{"C": 1931796, "M4": 59295, "Shell": 39878, "Makefile": 3071}` — an autotools-based C project (M4 for
`autoconf` macros, `Makefile`/`Makefile.am` for `automake`), no other language present. `src/` contains many
platform-conditional files (`netcpu_kstat.c`, `netcpu_osx.c`, `netcpu_perfstat.c`, `netsec_linux.c`,
`netsec_win.c`, etc.) — this shape (one shared core plus many OS-specific backends selected at configure/build
time) is the basis for the "platform-conditional code" checklist item in `references/review-checklist.md`; it
is inferred directly from the file layout, not from any review ever having flagged a cross-platform regression
here (none has, because no PR has ever been reviewed here at all).

## Bottom line

Every plank of a typical "mined maintainer voice" skill — merged/rejected PR patterns, named reviewers, a
documented style guide, a track record of what kinds of bugs slipped through — is absent for this fork. What
this skill instead grounds itself in is the two things that *are* independently verifiable: the literal
contents of the codebase (autotools + platform-conditional C, no tests, no prior CI), and the fact that this
review process itself is a brand-new addition rather than a continuation of an existing one.
