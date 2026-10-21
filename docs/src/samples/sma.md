# Moving-average crossover (sma_cross)

The implemented reference is `light-trader/guest/examples/sma-cross/src/lib.rs` at the [pinned compatibility baseline](compatibility.md). It is a **one-share, event-on-cross** teaching strategy,
not a full-equity target-position strategy.

## Current reference behavior

- Required parameters: `account`, `inst`, `calendar`, `entitlement`; unknown fields rejected.
- Timer: 15 minutes before the actual calendar close, including early closes.
- Query: 201 completed RTH daily bars with end at the current session open. Today's forming
  daily bar is excluded. Insufficient history skips.
- Compute previous 50/200 means from the first 200 closes and current means from all 201.
- If previous fast <= slow and current fast > slow, buy one share only when strategy position is zero.
- If previous fast >= slow and current fast < slow, sell the full positive strategy position.
- Submit a DAY market-on-close intent only if the strategy has no open orders.
- No new cross means no trade; starting midway through an established bullish regime does not buy.

```json
{"account":"paper","inst":"SPY","calendar":"us-equity","entitlement":"history"}
```

These are guest parameters only. `fast`, `slow`, risk allocation and target equity are not
configurable in this reference. Adding those creates a separately tested strategy version.

## Policy and acceptance limits

This reference submits an unbracketed MOC order. The existence of this demonstration is not
permission to bypass the engine's protection/risk policy. Paper deployment requires explicit
support for the account/instrument/order/exposure policy and actual MOC behavior. Do not claim
that a daily timer protects a position during an outage.

The guest relies on host strategy positions/open orders; S2 must qualify persistent decision
identity and restart behavior, partial/unresolved MOC orders, duplicate timers, restored positions
and rejected close orders. Use the same component for historical and decision replay. Compare
one-share trade quantities, not an unrelated full-investment backtest's P&L.
