# Platform Gotcha Catalog

A grow-over-time log of ad-platform and tooling gotchas that cost real
debugging time, each written up in the same shape: **symptom** (what you
actually observed), **misdiagnosis** (the plausible-but-wrong explanation
that ate the first chunk of time), **actual cause** (what was really going
on, and how to check for it going forward).

New entries welcome via PR — same three-part shape, no client or account
specifics, just the platform behavior.

---

## GAQL `campaign_shared_set` REMOVED-link false positive

**Symptom:** A query against Google Ads' `campaign_shared_set` resource —
used to check which negative-keyword or placement-exclusion shared sets
are linked to a campaign — returns rows for a shared set that was
deliberately unlinked from that campaign weeks ago. Downstream logic that
trusts "row present = still linked" treats the campaign as protected by a
list it no longer uses.

**Misdiagnosis:** Assumed to be a caching or propagation-delay issue on
Google's side — "give it a day, it'll clear." It doesn't clear, because
that's not what's happening.

**Actual cause:** GAQL's default query behavior over `campaign_shared_set`
does not filter out REMOVED-status links unless the query explicitly
constrains `campaign_shared_set.status`. A link that was removed still
comes back as a row; only its `status` field changed, not its presence in
the result set. Any consumer of this resource must filter on
`campaign_shared_set.status = 'ENABLED'` explicitly — there is no implicit
filtering of removed links, unlike some other Google Ads resources where
REMOVED rows are excluded by default.

---

## Fine-grained PAT URL form

**Symptom:** A fine-grained GitHub personal access token, generated
correctly with the right repository and permission scopes, gets rejected
or silently produces a 404-equivalent when used against an API endpoint
that worked fine with a classic PAT.

**Misdiagnosis:** Assumed the token's scopes were wrong, or that
fine-grained tokens don't support the endpoint being called at all.
Re-generating the token with broader scopes "fixes" it, masking the real
issue and widening the token's blast radius unnecessarily.

**Actual cause:** Fine-grained PATs are scoped to a specific resource
owner and repository set at creation time, and the token creation *URL*
itself needs to be generated in the context of the correct owner
(personal account vs. an org you belong to) — generating one from the
wrong context silently scopes it to repos you didn't intend, or to none
of the ones you need. Before re-scoping, verify the token's actual
attached repository list in the token settings page, not just the
permission checkboxes — the two are configured in different steps of the
creation flow and it's easy to get the resource-owner step wrong without
noticing.

---

## Redeploy ≠ rebuild

**Symptom:** A code fix is committed, the deploy pipeline reports success,
and the bug is still live in production — sometimes for hours, until
someone notices and manually forces a rebuild, at which point it's fixed.

**Misdiagnosis:** Assumed the fix itself was wrong, or that there's a
second code path serving the same behavior that didn't get patched. Time
gets spent re-reading the diff and searching for a duplicate implementation
that doesn't exist.

**Actual cause:** Several container-based deploy platforms distinguish
"redeploy" (restart the existing container image, no rebuild) from
"deploy" / "rebuild" (build a fresh image from the current source and
replace the running one). A pipeline configured to redeploy on push —
common when the platform's default trigger is "restart on git push" rather
than "rebuild on git push" — restarts the *old* image with the *old* code,
reports success because the restart itself succeeded, and never picks up
the new commit. Confirm which action a push actually triggers, not just
that the pipeline went green.

---

## API-version sunset rot

**Symptom:** A previously-stable integration against an ad platform's API
starts failing intermittently, then completely, with no code change on
your side and no obvious announcement you recall seeing.

**Misdiagnosis:** Assumed to be a transient outage or rate-limiting issue.
Retries and backoff get added, which sometimes appears to "fix" it because
the platform is still serving the sunset version to a shrinking population
during the deprecation window — masking the real deadline until it's
enforced for everyone.

**Actual cause:** Ad platform APIs version aggressively (Google Ads API
ships a new major version roughly every quarter and sunsets old ones on a
fixed published schedule) and a pinned client library or hardcoded
API-version header silently becomes unsupported the day the sunset date
hits. There is usually no runtime warning until the cutover — the version
either works or returns hard errors, with little gradient in between.
Treat API-version currency as a scheduled maintenance item, not a
reactive one: track the platform's published deprecation calendar and
bump the pinned version before the sunset date, not after the errors
start.
