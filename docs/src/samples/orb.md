# Opening range breakout (orb_sample)

The current reference is `light-trader/guest/examples/orb-sample/src/lib.rs`, at the [pinned compatibility baseline](compatibility.md). It is a public, unfiltered educational ORB inspired by
published opening-range research. Its futures adaptation and calculations below must not be
represented as an exact reproduction of a paper or a private strategy.

## Current reference behavior

| Item | Implemented behavior |
|---|---|
| Parameters | Required `account`, `inst`, `calendar`, `entitlement`; unknown fields rejected |
| Timer | Entry at calendar open + 305 seconds; close at actual calendar close - 10 seconds |
| Signal | First completed RTH 5-minute bar beginning exactly at session open; close > open buys, close < open sells; doji skips |
| Warm-up | Exactly 14 prior completed RTH daily bars, queried with end at current session open |
| Risk distance | 0.10 times arithmetic mean of the 14 daily high-low ranges; **not Wilder ATR or true range** |
| Size | Whole units rounded down under 1% configured-equity risk and 4x notional leverage; zero skips |
| Entry | Market native-bracket intent, reference price = opening-bar close, stop offset = risk distance, target offset = 10 times distance |
| Exit | Requests host `flatten` at the close timer; unsupported behavior returns an error |

```json
{"account":"paper","inst":"/MNQ","calendar":"us-equity","entitlement":"history"}
```

This is an illustrative guest-parameter object, not a complete deployable configuration. The
calendar must have reviewed sessions, the account's entitlements must match actual data sources,
and `/MNQ` needs instrument/contract metadata. Fixed 1%/4x/14/10 values are current code constants,
not accepted configuration knobs. Changes require a new documented strategy version.

The timer submits a market intent after the opening window. A `next_open` bar simulation fills
at the next eligible open after submission, not retroactively at 09:35:00. Record this distinction
when comparing against a research model with an earlier idealized entry.

## Protection and recovery

Native protection is submitted **with the entry** through `submit_bracket`. The guest does not
wait for a fill and then create unprotected child orders. The host owns broker-held protection,
fill-relative adjustment, partial-fill handling, order IDs, reconciliation and flatten safety.
Successful intent admission does not prove broker acceptance. Unqualified capabilities refuse.

The reference does not implement persistent per-session entry deduplication or a research-specific
late-entry cutoff. S2 must define and test those before repository paper acceptance: duplicate
entry timer, restart after admission, late/backfilled opening bar, unknown order outcome and
recovery past the allowed entry window. Do not assume a default no-op `on_feed` implements a
skip-day rule; host readiness and any additional strategy policy must both be tested.

No entry on missing opening history, doji or insufficient warm-up. Invalid/zero risk distance
fails explicitly. Add tests for short sizing, invalid metadata, early close, roll, stale inputs,
refused bracket/flatten and crash recovery. See [milestones](implementation.md).
