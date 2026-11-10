# PMG-003: Don't decide from one week

**Rule:** Compare this period to the trailing 6-8 periods, not to the one before it. Act only when it's outside that range.

## Why
On most accounts CPA and conversions move 20-30% week to week with nothing wrong. Auctions, seasonality and conversion lag all add noise. A one-period comparison turns that noise into a steady stream of "fix this" calls, and next week's swing back reverses them.

Hypothetical: CPA over the last 8 weeks ran between 80 and 115. This week it's 108, up 25% on last week's 86. Week over week, that's an alarm. Against the 8-week range it's an ordinary week, and the right call is to hold.

The exceptions are facts, not trends: overspend, broken tracking, a landing page that's down, a disapproved ad. Act on those immediately.

## What it looks like when an agent gets it wrong
- "Compare to the previous period" or "last 7 days vs prior 7 days" as the only input.
- "Hold if within normal variance" with no definition of normal variance.
- Pausing, rebidding or reallocating because of one week's move.

## Detection pattern
```
grep -rniE "previous (period|week)|prior 7|last_7_days|week[- ]over[- ]week|wow|normal variance" .
```
Then check whether the decision step uses a multi-period baseline.

## Fix template
> Build a baseline from the trailing 6-8 periods. Normal variance means inside that range. Only recommend a change when the current period falls outside it, or when the issue is a fact (spend, tracking, broken page, disapproval) rather than a trend.

## Fixed in
- [coreyhaines31/marketingskills #945](https://github.com/coreyhaines31/marketingskills/pull/945) (merged 2026-11-09)
