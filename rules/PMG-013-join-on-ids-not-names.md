# PMG-013: Join Google Ads resources on the right IDs

**Rule:** Join on resource IDs, never names, and know which IDs are only unique inside a parent.

## Why
GAQL joins that look right can quietly match the wrong rows. They don't error. They return plausible numbers attached to the wrong keyword, bid strategy or list. Four traps come up again and again:

1. **Names.** Campaign, ad group and label names aren't unique and can be renamed. A dict keyed by name merges two campaigns that share one, or loses history after a rename.
2. **Keyword criterion IDs.** `ad_group_criterion.criterion_id` is unique only within its ad group. The same keyword text in two ad groups can share a criterion ID. Key on (`ad_group.id`, `criterion_id`).
3. **Manager-owned portfolio bid strategies.** A portfolio strategy owned by a manager account isn't returned by `bidding_strategy` from the client account. Use `accessible_bidding_strategy`.
4. **Removed shared-set links.** `campaign_shared_set` keeps rows for links that were removed. Filter on `campaign_shared_set.status = 'ENABLED'` when asking which lists apply now. (Details: [platform gotcha catalog](../platform-gotcha-catalog.md).)

Hypothetical: a keyword report keyed on `criterion_id` alone. "running shoes" exists in a brand-adjacent ad group and a generic one with the same criterion ID. The report shows one row with the two ad groups' spend added together, and the bid change meant for one lands on both.

## What it looks like when an agent gets it wrong
- `{row.campaign.name: row for row in rows}`
- Keyword lookups keyed on `criterion_id` with no ad group.
- `FROM bidding_strategy` returning nothing for a campaign that clearly uses a portfolio strategy, followed by "no bid strategy set".
- "This campaign has a negative list attached" based on a removed link.

## Detection pattern
```
grep -rnE "\.name *[:\]]|by_name|criterion_id|FROM bidding_strategy|campaign_shared_set" .
```
Then check each hit for the matching trap: name-keyed dicts, criterion IDs without an ad group, `bidding_strategy` reads from a client account, `campaign_shared_set` with no status filter.

## Fix template
> Key every join on resource IDs or resource names, not display names. Key keywords on (ad group ID, criterion ID). Read portfolio strategies through `accessible_bidding_strategy`. Filter `campaign_shared_set` on status when asking what applies now.

## Fixed in
None merged yet.
