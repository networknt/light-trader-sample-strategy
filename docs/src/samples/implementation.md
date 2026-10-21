# Implementation milestones

Deliver the template in reviewable milestones using the [ownership decision](compatibility.md)
and [architecture contract](architecture.md). See the overview for current status.

| Milestone | Deliverables | Acceptance |
|---|---|---|
| S0: compatibility baseline | Pin engine/SDK/WIT/toolchain; consume the canonical upstream examples without copies; inventory existing reference behavior | Contract matrix and exact source revisions reviewed; no invented API or release tags |
| S1: build template | Pin-reading checkout/build scripts, custom-strategy starter, synthetic fixtures and artifact manifests | Canonical samples build from the pinned upstream guest workspace; a separate custom starter resolves SDK/WIT assets and builds successfully |
| S2: upstream lifecycle hardening | Contribute parameter validation, session/action deduplication, state schema and restart policy upstream; advance the template pin after acceptance | Duplicate timers, restart between admission and state save, pending/partial/unresolved orders, malformed state and feed recovery tested with real host/OMS where needed |
| S3: offline replay | Separate replay configs, fixed synthetic calendars/instruments, expected calls/trades and reports | Same component hashes, repeated results, no future data, explicit fill/cost model and zero unexplained parity differences |
| S4: CI and artifacts | Native plus compiled-component checks, formatting/lint, docs build, import inspection and compatibility manifest | Clean checkout builds without private data/accounts; no broker network access; supported host/version matrix recorded |
| S5: optional stack template | Isolated offline default and explicit paper profile; pinned accepted service images; gateway integration | Offline startup/replay, private networking, secrets handling, health/readiness and rollback checked; paper profile refuses missing qualifications |
| S6: paper qualification | Supervised sample runs, daily historical/decision comparisons, restart/outage scenarios | Phase 10's required paper days and broker gates remain satisfied separately; failed/incomplete days retained |

## Proposed layout

```text
light-trader.rev                   single upstream source pin
.upstream/light-trader/             ignored pinned checkout, canonical sample sources
templates/custom-strategy/          starter for a new user-owned component
config/replay/                      offline configs and synthetic session metadata
config/paper/                       disabled examples, no credentials
testdata/synthetic/                 generated fixtures and expected outputs
scripts/                           build, inspect, replay, compare, validate
artifacts/                         ignored local WASM/manifests/reports
deploy/                            future Compose profiles and gateway route profile
docs/                              this book
```

Do not move the broker/risk/engine implementations into this workspace. Extract reusable numeric
helpers only where needed; shared public helpers belong in the SDK with upstream tests. Do not copy
private rules, logs, research figures, account metadata or licensed data into public fixtures.

## Required behavior tests

- ORB: absent/wrong opening bar, insufficient prior sessions, doji, invalid distance/price,
  sizing rounded down, long/short bracket offsets, zero quantity, duplicate entry decision,
  late/recovered event after entry cutoff, early close and refused bracket/flatten capabilities.
- SMA: exactly 200 versus 201 completed sessions, equality/cross direction, no repeated order
  while pending, existing position, fractional/negative anomalies, early-close timer and MOC rejection.
  Keep the one-share reference semantics until a separately named strategy version changes them.
- Both: account/spec mismatch, entitlement denial, changing contracts, delayed inputs, budgets,
  unknown state version, restart, rejected/partial/unresolved orders and fail-closed recovery.

Pin expected outputs from an independently explained rule calculation. Do not regenerate goldens
automatically after behavior changes. Record why a golden changed and whether the strategy version
or only an implementation defect changed. Reference sample limitations are not acceptance waivers.

## Concurrency and custom-plugin acceptance

Add a same-module/two-instance test proving independent state and non-overlapping callbacks per
instance. Run a healthy strategy beside a budget-exhausting or slow-host-call strategy and measure
bounded latency/queue growth. Exercise shared-account concurrent entry admission through the real
engine/OMS; two independent guests must not both spend the same capacity. Keep deterministic
historical replay tests separate from live scheduling tests.

Use the separate custom-strategy starter as the external developer test: pinned SDK/WIT,
clean build, import inspection, actual host load and replay without internal server dependencies.
These supplement S2–S4; they do not qualify broker behavior or hostile multi-tenant isolation.

## Delivery report

Report component/engine/SDK/WIT hashes, exact commands and counts, fixtures, timing policy, native
versus actual-WASM results, local versus remote CI and outstanding paper gates. A successful
simulated bracket is never Q0–Q8 evidence. Do not publish images/modules from this design task.
