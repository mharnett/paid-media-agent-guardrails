# Paid Search Agent Guardrails

Agents are writing to ad accounts everywhere — pausing campaigns, adjusting
bids, applying platform recommendations, editing keywords — often on a
schedule, with no human in the loop for every change. The platforms
(Google Ads, Microsoft/Bing Ads, Meta Ads) are converging on the same
agent-assisted and native "auto-apply" surfaces, and the tooling built on
top of them is converging on the same failure classes: a scoring formula
that silently collapses to one bucket, a mutation with no audit trail, an
API contract that quietly changes shape between versions.

This repo is a running collection of the guardrails that catch those
failures before they reach a live ad account — patterns, gotchas, and
methodology, written up as they're found. It is not tied to any one
platform, agency, or account. The failure classes are global; the fixes
should be too.

## What's here

- **[shape-anchor-testing.md](shape-anchor-testing.md)** — why unit tests
  alone don't catch bugs in scoring/classification/pipeline code, and the
  anchor + shape testing pattern that does.
- **[label-every-mutation.md](label-every-mutation.md)** — treating a
  machine-readable label as the audit and rollback primitive for every
  agent-driven mutation.
- **[platform-gotcha-catalog.md](platform-gotcha-catalog.md)** — a
  grow-over-time catalog of ad-platform API gotchas, each written up as
  symptom → misdiagnosis → actual cause.
- **[guard-function-pattern.md](guard-function-pattern.md)** — turning a
  prose safety rule into a pure guard function, then wiring it into a
  pre-commit hook or CI check so it can't be skipped.

## Who this is for

Anyone building or operating agents (LLM-driven or otherwise) that read
from or write to paid search / paid social ad platforms — whether that's a
one-off script, a scheduled job, or a full agentic workflow with
human-in-the-loop approval. The lessons here come from running exactly
that kind of system in production, generalized and stripped of any
account-specific or client-specific detail.

## Contributing

This is currently a personal running log. Issues and PRs describing a new
gotcha, a sharper guard function, or a correction are welcome.

## License

[MIT](LICENSE)
