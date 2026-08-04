# Guard Function Pattern

*Optional for v1 — included here because the pattern is small and the repo
already needed an example of it in `tooling/guardrails-repo-standup/` for
its own sanitization and doc-set checks.*

A safety rule that lives only in a doc or a memory file is a rule someone
has to remember to apply. It gets followed the first few times, then
skipped under deadline pressure, then forgotten entirely once the person
who wrote it moves on. The fix is to stop treating it as a rule and start
treating it as a function with a boolean return value, wired into
something that runs automatically.

## The three-step conversion

**1. Prose rule.** Write the rule the way you'd explain it to a
colleague: "every mutation to a live ad account must carry an audit
label" or "a public doc in this repo must never mention a real client
name." This is where the rule starts, and it's fine for it to stay
somewhat informal here — this is the version a human reads to understand
*why* the rule exists.

**2. Pure guard function.** Extract the checkable part of the rule into a
function with no side effects: given the artifact being checked (a
diff, a set of filenames, a piece of text), return a pass/fail verdict
plus enough detail to act on a failure. Pure functions are the ones worth
writing this way because they're trivially testable — no mocking a
platform API, no fixture describing an entire deploy pipeline, just input
in, verdict out. This repo's own `sanitize.ts` (`scanForClientNames`) and
`docSet.ts` (`validateDocSet`) are both this pattern applied to this
repo's own two rules: no client names, and a complete-enough doc set.

**3. Pre-commit / CI hook.** Wire the guard function into something that
runs without being asked — a pre-commit hook for fast local feedback, a CI
check as the backstop that can't be skipped by a `--no-verify` in a hurry.
The guard function itself doesn't change between the two; only the
trigger does. A hook that shells out to the same function gives identical
behavior locally and in CI, which matters — a rule that fails differently
in the two places just moves the argument about whether it's real to
review time.

## Why this ordering, and not "just write the CI check"

Skipping straight to a CI check without the intermediate pure function
tends to produce a check that's entangled with the CI environment —
reading a `git diff` in a specific format, assuming a checked-out working
tree, hard to run locally in isolation. Extracting the pure function first
forces the actual rule to be expressed independent of *how* it gets
invoked, which is what makes it possible to unit-test it with hand-picked
inputs (anchor tests) and to run it in more than one place without
duplicating logic.

## When this pattern doesn't fit

Not every rule reduces to a pure function. "This campaign structure looks
wrong for this account's goals" is a judgment call, not a checkable
invariant — trying to force it into a guard function usually produces
either a function so narrow it misses real violations, or one so broad it
false-positives constantly. Reserve this pattern for rules that really are
checkable from the artifact alone: presence/absence, format, pattern
matching, structural invariants. Judgment calls stay judgment calls, and
get a human reviewer instead of a hook.
