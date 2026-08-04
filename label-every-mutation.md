# Label Every Mutation

If an agent can pause a campaign, adjust a bid, apply a platform
recommendation, or edit a keyword, then at some point a human is going to
ask two questions: *what changed, and how do I undo it?* Logs answer the
first question until they roll off retention. They rarely answer the
second cleanly, because "undo" requires knowing exactly which entities a
given run touched — not approximately, exactly.

## The primitive: a machine-readable label on every mutation

The fix is boring on purpose: every entity an agent creates, pauses,
enables, or edits gets a label in a fixed format —

```
claude-MM-DD-YY
```

(or the equivalent for whichever agent/tool is doing the mutating — the
format matters less than the discipline). Apply it at creation time if the
platform's API supports labeling in the same call; otherwise apply it
immediately after, before the mutation is considered complete. No
exceptions for "this one's obviously safe" — the whole point is that the
label doesn't require the human doing the audit to trust that judgment
after the fact.

## Why a label beats a log

A log tells you an agent ran and roughly what it did. A label, applied to
the entity itself:

- **Survives outside the logging system.** Query the ad platform directly
  — "show me everything with label `claude-06-14-26`" — and get the exact
  entity set, even if the run's logs have expired or the logging pipeline
  had an outage that day.
- **Composes across runs.** A batch of edits from three different days
  shows up as three distinct label values, so a rollback of "just
  Tuesday's batch" is a filter, not an archaeology project.
- **Is the audit trail, not a proxy for one.** "Who/what changed this
  entity and when" is answerable by reading the entity, not by
  cross-referencing a timestamp against a log line and hoping the clocks
  agree.

## Where this breaks if you're not careful

- **Partial application.** If the API call that creates an entity doesn't
  support labeling atomically, and the follow-up labeling call fails or is
  skipped, you get an unlabeled mutation indistinguishable from manual
  human action. Treat "label applied" as part of the mutation's success
  condition, not a best-effort afterthought.
- **Format drift.** `claude-06-14-26` vs `claude-6-14-2026` vs
  `agent-2026-06-14` across different tools or different points in a
  project's history makes "find everything from this run" a regex
  exercise instead of an exact match. Pick one format, document it, and
  don't let it drift per-integration.
- **No coverage for reads.** This pattern is for mutations only. Don't
  over-apply it to read-only operations — the audit need doesn't exist
  there, and labeling reads just adds noise to the entity's label history.

## The test

Before treating any agent-driven mutation path as done, ask: *if this ran
unattended overnight and something looked wrong in the morning, could I
find and revert exactly what it touched — no more, no less — using only
the platform's own UI or API, with no access to logs?* If the answer
requires guessing based on timestamps, the labeling discipline isn't
finished yet.
