# PMG-006: Ad Strength is not a performance KPI

**Rule:** Judge responsive search ad copy by performance (expected CTR and conversion rate, weighted by impressions), not by Ad Strength.

## Why
Ad Strength scores how varied and keyword-relevant an ad's assets are. Google says it isn't used to calculate Ad Rank, Quality Score or auction wins ([About Ad Strength for responsive search ads](https://support.google.com/google-ads/answer/9921843)). Chasing "Excellent" pushes toward more, and more generic, headlines. That can lower CTR while the score goes up.

Hypothetical: two ads in the same ad group. Ad A is rated "Average" and takes 70% of impressions at a 6% CTR. Ad B is rated "Excellent" and takes 30% at 4%. A rule that says "replace anything below Good" swaps out the better ad.

## What it looks like when an agent gets it wrong
- "90%+ of RSAs rated Good or Excellent" as a success metric.
- "Start every creative refresh from ad strength data."
- Pausing or rewriting ads because their Ad Strength is "Poor" or "Average".

## Detection pattern
```
grep -rniE "ad[_ ]strength" . | grep -viE "not (a|an|used)|diversity|isn't"
```
Then check whether the hit uses Ad Strength as a goal, a KPI or a reason to change an ad.

## Fix template
> Use Ad Strength as a checklist for asset variety, not as a goal. Judge copy on impression-weighted CTR and conversion rate against the other ads in the same ad group, over a window long enough to be past noise.

## Fixed in
None merged yet. Open: [msitarzewski/agency-agents #1232](https://github.com/msitarzewski/agency-agents/pull/1232).
