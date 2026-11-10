# PSG-004: Don't filter historical metrics on current status

**Rule:** When pulling metrics over a date range, don't filter on status. Status is today's value, not the value during the range.

## Why
`campaign.status = 'ENABLED'` or `!= 'REMOVED'` on a performance query drops every campaign (or ad group, ad, keyword) that was paused or removed after it spent. The query runs fine. The totals are just lower than the account's, and nothing flags it.

Hypothetical: a campaign spent 4,000 in the first half of the month and was removed mid-month. A month-to-date report filtered on `!= 'REMOVED'` leaves out that 4,000 and its conversions. CPA looks better than it was, and the totals don't reconcile with billing.

## What it looks like when an agent gets it wrong
- `WHERE campaign.status != 'REMOVED' AND segments.date DURING LAST_30_DAYS`
- `WHERE ad_group_criterion.status = 'ENABLED'` on a keyword performance query.
- Listing "active campaigns" first, then pulling metrics only for those IDs.

## Detection pattern
```
grep -rniE "status *(=|!=) *'(ENABLED|REMOVED)'" . | grep -iE "segments\.date|DURING|BETWEEN|date_clause"
```
Then check the hits for metrics in the same query.

## Fix template
> Don't filter on status in date-ranged metric queries. Select the status field instead. To hide removed entities with no activity, drop rows with zero impressions after the query (GAQL has no OR, so this can't go in the WHERE clause). Filter on status only when the question is what is live now.

## Fixed in
- [kLOsk/adloop #128](https://github.com/kLOsk/adloop/pull/128) (merged 2026-11-08)
