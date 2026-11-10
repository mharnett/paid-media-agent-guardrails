# PMG-005: Never truncate ad copy

**Rule:** If a headline or description is over the limit, reject it or rewrite it. Never slice it to fit.

## Why
Responsive search ad headlines are capped at 30 characters, descriptions at 90 and display paths at 15. Code that slices text to fit (`text[:30]`) always produces a valid ad. It just ships half a sentence. The API accepts it, nothing errors, and the broken copy serves until someone happens to read it.

Hypothetical: a description written as "Protect sensitive data across cloud, endpoint and email with one policy" plus a few more words gets cut at 90 and serves as "…across cloud, endpoint and". Nobody reviewed that text, because nobody wrote it.

## What it looks like when an agent gets it wrong
- `ad_text_asset.text = headline[:30]`
- `description.slice(0, 90)` or `substring(0, 90)` before a mutate call.
- A helper named `_truncate_headline`, with a test that locks the truncation in.
- An agent told to "make it fit" that trims the last words instead of rewriting.

## Detection pattern
```
grep -rnE "\[:(15|30|90)\]|slice\(0, ?(15|30|90)\)|substring\(0, ?(15|30|90)\)|truncate" .
```
Then check whether the hit sits on ad text before a create or update call.

## Fix template
> Validate length before the API call. If any headline, description or path is over its limit, fail with an error that names each item and its length, and leave the rewrite to the writer (human or agent). Write templates that fit; if a generated line overflows, drop it rather than cut it. Note that Google counts some wide characters (such as CJK) as two.

## Fixed in
None merged yet.
