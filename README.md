## What I work on

I build and audit development infrastructure for AI coding agents — the environment
around the agent rather than the agent itself: hooks, guards, reproducible containers,
and the checks that catch a tool lying about its own results.

Most of what I publish comes out of debugging something that was quietly wrong for a
while. That is the thread: **a green check mark is a claim, and claims should be
falsifiable.**

### Bugs I've reported upstream

- [anthropics/claude-code#78788](https://github.com/anthropics/claude-code/issues/78788) —
  ESC keypresses intermittently dropped under load; split escape sequences leak literal
  fragments into the prompt (tmux + Terminal.app)
- [stablyai/orca#9563](https://github.com/stablyai/orca/issues/9563) —
  headless `orca serve` under a LaunchAgent invokes quit-and-install, disconnects daemon
  clients, and never comes back up on the new version

### Projects

- **[bats-ghost-test-check](https://github.com/KinohTaGo/bats-ghost-test-check)** —
  finds bats tests that are declared, counted in the TAP plan, and never executed.
  Root cause is a locale-dependent `[[:alnum:]]` in bats-core's test-name encoder: under
  a UTF-8 locale, some non-ASCII test names silently stop running. Found it after
  discovering 85 tests in my own suite had not run in an unknown number of weeks.
- **[cc-cleanroom](https://github.com/KinohTaGo/cc-cleanroom)** —
  digest-pinned, multi-arch container for reproducible Claude Code CLI runs, with a
  runtime verifier so a drifted container fails loudly instead of returning a plausible
  answer. Provenance attestations and SBOM published.
- **[claude-code-internals](https://github.com/KinohTaGo/claude-code-internals)** —
  field notes on observed CLI behaviour, each one tagged with the version it was verified
  against. Includes the notes where my own earlier conclusion turned out to be wrong,
  because that is usually the useful part.
- **[staleness-resistant-docs](https://github.com/KinohTaGo/staleness-resistant-docs)** —
  a documentation pattern for long-lived project instructions: every factual claim
  carries the date it was measured and the command to re-measure it, and claims that went
  stale record what the staleness actually cost.

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
