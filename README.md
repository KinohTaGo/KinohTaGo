## What I work on

I build and run development infrastructure for AI coding agents — the environment
around the agent rather than the agent itself: hooks, guards, scheduled self-checks,
and the checks that catch a tool lying about its own results.

Most of what I publish comes out of debugging something that was quietly wrong for a
while. That is the thread: **a green check mark is a claim, and claims should be
falsifiable.**

> 2025 年 5 月から、Claude Code をローカル LLM などと共に自宅の Mac mini と MacBook 等で
> 運用するための基盤インフラを構築・運用しています。下の公開リポジトリは、その運用の中で
> 作った道具と文書を切り出したものです（基盤そのもののコードは非公開）。

### At a glance

| | What it is | Status |
|---|---|---|
| **[bats-ghost-test-check](https://github.com/KinohTaGo/bats-ghost-test-check)** | Two checks that find bats tests which are declared and counted, but never run | v0.1.0 · the root cause I reported was fixed upstream (2026-09-12) |
| **[staleness-resistant-docs](https://github.com/KinohTaGo/staleness-resistant-docs)** | A pattern and a checker: every factual claim in project docs carries the date it was measured and the command to re-measure it | v0.2.0 · checker with tests |
| **[agent-cleanroom](https://github.com/KinohTaGo/agent-cleanroom)** | Reproducible, network-isolated runs of coding agents against local LLMs or APIs, with a verifiable record of every run | 0.1.0, pre-release — published early so the design can be read |
| **[claude-code-internals](https://github.com/KinohTaGo/claude-code-internals)** | Field notes on Claude Code CLI behaviour, each tagged with the version it was verified against | documentation |
| **The infrastructure** ([below](#the-infrastructure-behind-it)) | Where the repositories above came from | private — described here, code not published |

### The infrastructure behind it

Since May 2025 I have been building and operating the infrastructure for running
Claude Code — together with local LLMs — on a Mac mini and a MacBook at home.
Each project runs in its own long-lived agent session, and a session delegates to
teams of parallel agents. Most of the work is keeping that honest:

- **Parallel agents with a gate.** Before a new agent starts, a guard reads the
  account's usage and the recent spawn count on the machine, and refuses when either
  is near its limit.
- **Sessions that message each other.** Sessions on both machines exchange messages
  through a git-backed relay, so warnings and handoffs survive restarts and reach the
  other machine.
- **A shared working tree, used safely.** Several agents commit to the same checkout.
  Agents may push documentation; a pre-push hook refuses any code change that lacks a
  per-commit approval marker, and the rule is that the marker is added only after I
  approve that change.
- **A pre-push static gate.** Lint, syntax checks for scripts and launchd job
  definitions, and a reconciliation of declared versus executed test counts.
- **Scheduled self-checks.** launchd jobs that confirm hooks and links still resolve
  and report drift between what is deployed and what is in the repository.
- **Shared memory for agents** in a separate git repository, read at the start of each
  session.

Measured 2026-11-08 on the main repository: 3,273 commits; 194 bats and 90 pytest
test files (file counts, not test counts).

### Bugs I've reported upstream

Status checked 2026-11-08.

- [bats-core/bats-core#1233](https://github.com/bats-core/bats-core/issues/1233) —
  tests with non-ASCII names are silently never executed under a UTF-8 locale; the TAP
  plan still counts them, so a suite passes by not running. Found 85 such tests in my own
  suite. **Fixed:** PR [#1236](https://github.com/bats-core/bats-core/pull/1236) made the
  test-name encoding locale-independent, merged 2026-09-12 (not yet in a release; the
  latest is v1.14.0).
- [anthropics/claude-code#83326](https://github.com/anthropics/claude-code/issues/83326) —
  the shadowed `grep` silently skips gitignored files, so a secret scan reports clean
  while missing `.env` and `*.log` — and exits 0. Labelled *reproduced* by a maintainer;
  later closed automatically as stale (2026-10-19).
- [anthropics/claude-code#78788](https://github.com/anthropics/claude-code/issues/78788) —
  ESC keypresses intermittently dropped under load; split escape sequences leak literal
  fragments into the prompt (tmux + Terminal.app). Closed automatically as stale while
  labelled *needs-repro* (2026-08-24).
- [stablyai/orca#9563](https://github.com/stablyai/orca/issues/9563) —
  headless `orca serve` under a LaunchAgent invokes quit-and-install, disconnects daemon
  clients, and never comes back up on the new version. **Fixed:** PR
  [#9634](https://github.com/stablyai/orca/pull/9634), merged 2026-07-22.

### How I work

Three habits, learned the hard way:

**Measure before asserting.** If a claim about the current state can be checked with
`grep`, `ls`, or a five-second command, I check it before writing it down. If it can't be
checked, it gets labelled as an assumption rather than a fact.

**Break the check to prove it checks.** A passing test proves nothing until you have
watched it fail for the right reason. I have shipped two "green" test suites that were
green because they were not testing anything.

**Record the correction, not just the conclusion.** When an earlier explanation turns out
to be wrong, the wrong one stays in the document with a note. Otherwise the next person —
usually me — re-derives it from scratch.

---

<sub>Interested in agent infrastructure, reproducibility, and the failure modes of
tooling that reports success.</sub>
