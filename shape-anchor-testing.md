# Shape + Anchor Testing

Unit tests verify `f(x) == y` for a chosen `x`. That's necessary, but for
any code that produces structured or aggregated output — a scoring
formula, a classifier, a pipeline stage, anything that emits a
distribution, a schema, or an ordered collection — it is not sufficient.
A unit test can pass on every hand-picked input while the *composition* of
those inputs is broken: a config section silently absent, a weight that's
never applied, a bucket that dominates every real-world distribution even
though every individual case looks correct in isolation.

## The incident

A B2B security client's offline-conversion value pipeline used an
8-multiplier formula to score conversion value based on account
attributes. On the runtime install, one config section was silently
absent — no error, no warning, just a missing key that a default
swallowed. The effect: the 8-multiplier formula collapsed to one dominant
value bucket. Every conversion got scored the same way, regardless of the
attributes that were supposed to differentiate it.

Unit tests were green for eight straight days. Each one checked
`f(specific_input) == expected_output` for a handful of hand-picked cases,
and every one of those cases happened to route through paths that didn't
expose the missing config. The bug surfaced only when someone manually
inspected the production histogram of scored values and noticed it was a
single spike instead of a distribution.

## What would have caught it

Two layers, added alongside (not instead of) unit tests:

**Anchors** — precise behavioral invariants on hand-picked inputs, phrased
as "category X → property Y, regardless of every other dimension." Use
exact equality where the design is sharp (this input *must* map to this
exact output), and range-based assertions where the design is
calibration-tunable (this input must land in this band, even if the exact
number is expected to shift as weights get retuned).

**Shape** — assertions on the aggregate output across a representative
input range. Two forms work well:

- A distribution snapshot compared against a committed golden file, with a
  tolerance band (so legitimate retuning doesn't break the test, but a
  structural collapse does).
- Property-based assertions on aggregate measures: cardinality ≥ N (the
  output actually uses more than one bucket), no single bucket > X% of
  total volume, the schema's key set matches what's expected, ordering
  invariants hold.

A shape test on that offline-conversion pipeline — "no single value bucket
should hold more than, say, 60% of scored conversions" — would have failed
on day one, not day eight.

## The question to ask before shipping test coverage

*What bug class would unit tests, by construction, not catch here?*

If the answer is "a wiring, composition, or config bug that makes every
individual unit test pass while the aggregate output is wrong," that's the
signal to add an anchor or shape test, not another unit test with a
different `x`.

## When a shape test is missing after a production bug

If a bug like this reaches production and no shape test existed for that
code path, treat the *absence* as a coverage gap to close — not just the
specific regression to fix. The next config key that goes missing, or the
next multiplier that silently stops applying, should fail a shape test
before it fails a customer.
