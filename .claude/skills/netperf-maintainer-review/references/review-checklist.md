# Review checklist (generic, grounded in codebase shape — not in any mined review precedent)

None of these items come from a real reviewer having flagged them before on this fork — see
`grounding-note.md` for why there is no such precedent to draw on. Each item is instead grounded in a concrete,
verifiable fact about the codebase itself (its build system, its file layout, or the simple absence of CI).
Apply judgment about relevance per-PR; don't force-fit every item onto every diff.

## 1. Autotools build-system changes

netperf is built with autoconf/automake (`configure.ac`, `Makefile.am`, `acinclude.m4`, `m4/`). If a PR edits
any of these:
- Generated artifacts (`configure`, `Makefile.in`, `aclocal.m4`) are usually NOT meant to be hand-edited or
  committed alongside source changes — check whether the PR regenerates them via `autoreconf`/`autogen.sh` or,
  worse, hand-patches a generated file directly (a real risk: hand-edits to generated files silently get
  clobbered or drift the next time someone regenerates them).
- A new source file needs to actually be wired into the relevant `Makefile.am`'s `_SOURCES` list, or it will
  build locally (if someone's stale `Makefile` still references it) but fail on a clean checkout.

## 2. Platform-conditional code

`src/` contains many OS-specific backends behind naming conventions like `netcpu_<platform>.c`,
`netsec_<platform>.c`, `netdrv_<platform>.c`, `netfirewall_<platform>.c`, `netrt_<platform>.c` — this is a
one-core-plus-many-backends design, each backend selected at configure time for its target OS.
- A change to a *shared* header (`netcpu.h`, `netsh.h`, `hist.h`) or to `netlib.c`/`netperf.c`/`netserver.c`/
  `netsh.c` (the platform-independent core) needs to be checked against how each backend uses the changed
  interface — this repo's CI (as of this PR) only ever builds on whatever single OS the Actions runner is, so
  a change that's fine on Linux can silently break a `#ifdef`-gated path for another OS with no CI signal at
  all.
- A change confined to a single `_<platform>.c` file is lower-risk by construction — it can't affect other
  backends' compilation, only its own platform's behavior.

## 3. Basic C correctness — because nothing else will catch it

There is no test suite and, as of this PR, no CI at all. Flag on sight, don't assume something downstream
will:
- Unchecked return values from `malloc`/`socket`/`recv`/`send`/etc., especially in code paths that run in a
  benchmark's hot loop.
- Buffer-size mismatches in `sprintf`/`strcpy`/`memcpy`-family calls, and any new fixed-size buffer that takes
  attacker- or user-controlled length input.
- Obvious signedness/overflow issues in anything computing byte counts, packet sizes, or timing — this is a
  benchmark tool, so silently wrong arithmetic here corrupts the very numbers netperf exists to produce, which
  is a worse failure mode than a crash.

## 4. CLI / flag surface changes

netperf's command-line surface is large and documented mainly in the `netperf.1`/manual, not machine-checked
anywhere.
- A new flag should be visible in the tool's own `-h`/usage output, not only implemented and documented
  externally — an undiscoverable flag is effectively dead code for most users.
- Check for a flag letter/name collision against the existing option-parsing switch — an accidental shadow of
  an existing single-letter flag is easy to introduce and easy to miss without a test asserting the option
  table.

## 5. Scope and blast radius

Given this fork's built-in purpose (mirroring/pinning a netperf revision for benchmarking use elsewhere in the
org, per the grounding note), a large, sweeping change to core benchmark logic is worth more scrutiny than a
narrow one — the fork's likely consumers depend on netperf's numeric output staying stable and comparable
across revisions, so a change that alters measured behavior (not just adds a platform backend or fixes a
build issue) deserves an explicit call-out even if it's otherwise well-written.
