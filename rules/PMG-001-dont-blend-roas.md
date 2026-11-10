# PMG-001: Don't blend ROAS across brand and non-brand, or retargeting and prospecting

**Rule:** Split brand from non-brand search, and retargeting from prospecting, before comparing ROAS or moving budget.

## Why
Brand search and retargeting mostly reach people who were already coming. Their ROAS looks great because the platform takes credit for sales that would have happened anyway. Blend them with non-brand or prospecting and the average says the account is healthier than it is. Rank by it and budget flows toward the campaigns that add the least.

Hypothetical: brand at 8.0 ROAS, non-brand at 1.5. Blended, the account shows about 3.0 and "looks fine". An agent told to cut the weakest campaign and fund the strongest moves money from non-brand into brand, which can't spend more because brand demand is capped by how many people search the name.

Treat brand and retargeting results as mostly non-incremental unless a holdout or lift test says otherwise.

## What it looks like when an agent gets it wrong
- "Allocate budget proportional to ROAS" across all campaigns, or per platform.
- "Shift 20% from the lowest-ROAS platform to the highest."
- "Look at blended CAC" with no segmentation step first.
- Reporting brand CPA as a win.

## Detection pattern
In skill or agent prompts, and in code that ranks campaigns:
```
grep -rniE "blended (roas|cac|cpa)|highest[- ]roas|avg_?roas|sort.*roas" .
```
Then check whether anything splits brand/non-brand (campaign name or a brand keyword list) and retargeting/prospecting (audience type) before the ranking.

## Fix template
> Before comparing performance or shifting budget, segment search into brand and non-brand, and display/social into prospecting and retargeting. Judge each segment against its own target. Brand and retargeting returns are mostly not incremental, so don't move budget into them on ROAS alone; use a holdout or lift test to size them.

## Fixed in
- [davepoon/buildwithclaude #221](https://github.com/davepoon/buildwithclaude/pull/221) (merged 2026-07-06)
- [OpenClaudia/openclaudia-skills #11](https://github.com/OpenClaudia/openclaudia-skills/pull/11) (merged 2026-09-11)
- [coreyhaines31/marketingskills #401](https://github.com/coreyhaines31/marketingskills/pull/401) (merged 2026-10-03)
