# PMG-016: Split brand from non-brand search before it goes into a media mix model

**Rule:** In a media mix model (MMM), brand search and non-brand search are separate channels. Usually brand search is left out of the media inputs entirely.

## Why
Non-brand search buys demand you wouldn't otherwise reach. Its spend responds to bids and budget, so it behaves like a media input.

Brand search mostly captures demand that other channels already created. Its spend follows that demand: more people search the brand name, so there are more brand clicks and more brand spend. That makes it closer to an outcome than an input. Put it in the same column as non-brand and the model sees search spend rising with sales, gives search the credit, and takes it away from the upper-funnel channels that drove the searches. The budget optimizer then moves money into search.

This is the modeling version of [PMG-001](PMG-001-dont-blend-roas.md).

Hypothetical: a TV flight lifts brand searches 40%. Brand search spend rises with it. In a blended `google_ads_spend` column, the model credits search for the sales lift, and TV's estimated effect shrinks.

## What it looks like when an agent gets it wrong
- Ad-platform data pivoted on platform, producing one `google_ads_spend` channel.
- "Filter by media type and market first" advice that never mentions brand versus non-brand.
- Brand search spend used as a media input with no note on why.

## Detection pattern
```
grep -rnE "platform_col|groupby\(.?platform|google_ads_spend|search_spend" .
```
Then check whether anything splits brand from non-brand (a campaign-name rule or a channel column) before the data reaches the model.

## Fix template
> Derive a channel column from campaign names (brand vs non-brand, using a rule the user supplies) and pivot on that instead of platform. Treat brand search as a separate channel, or leave it out of the media inputs and explain why.

## Fixed in
- [pymc-labs/pymc-marketing #3338](https://github.com/pymc-labs/pymc-marketing/pull/3338) (merged 2026-11-12): a brand/non-brand split step in the Fivetran connectors notebook and a note in the loader's docstring
