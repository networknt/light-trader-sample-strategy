# Testing a strategy

Keep five evidence levels separate: native callback logic, actual component/host behavior,
historical simulation, journal decision parity and broker paper acceptance.

## 1. Native and actual WASM

Use SDK `FakeHost` with synthetic sessions, known-at bars, instruments, open orders and scripted
action results. Assert decisions and errors, not merely that a callback returns. Build the same
crate as a component and run it under `lt-wasm-host`/`lt-engine` for real budgets, WIT validation,
state, risk and journal tests. Test restart and duplicate actions at the host boundary.

See [Writing a strategy](writing.md) for current build commands in the Light-Trader checkout.

## 2. Historical replay

The current binary uses `run`, `compare` and `journal` subcommands. Its Cargo package is
`lt-replay-cli`; it has a dedicated offline config schema, not a trade-server config. Commands
below run from a pinned **Light-Trader checkout** after building components and preparing an
isolated database with the required migrations, synthetic/history inputs and session metadata:

```sh
cargo run --locked -p lt-replay-cli -- run --help
cargo run --locked -p lt-replay-cli -- run   --config /path/to/offline-replay.toml --strategy orb_sample   --from 2026-01-02 --to 2026-02-01 --source bars --fill-model next_open   --out /tmp/orb-sample-replay
```

Replace the config path and dates with prepared inputs; the sample-specific offline config is an
S3 deliverable. Use `light-trader/config/replay.example.toml` and `bins/lt-replay/README.md` as the
schema/invocation authority. Credentials come from the config's environment-variable name,
not command-line URLs. Preserve reports in a fresh directory; do not use `--force` by default.

Dates are exchange-local; `--to` is exclusive. Warm-up history is queried before the period but
not delivered as simulated trading events. Record data revision, calendar, source, adjustment,
costs and fill model. `next_open` must not fill before submission. `same_bar_open` is explicitly
approximate and not an acceptance shortcut. Tick replay requires `--source ticks --fill-model ticks`
and the appropriate DB/Iggy archive transport. No final flatten is invented for open positions.

## 3. Independent rule comparison

```sh
cargo run --locked -p lt-replay-cli -- compare   --trades /tmp/orb-sample-replay/trades.csv --expected /path/to/reference.csv   --tick-size 0.25 --price-ticks 1 --r-tolerance 0.01   --out /tmp/orb-sample-comparison.csv
```

The shown tick size is an example: use the instrument/contract's actual metadata and predeclared
tolerances. Use the comparator's documented CSV schema. Keep default zero allowed mismatches;
review missing/extra trades, dates, sides, quantities, price and R differences individually.
Do not call same-code-generated golden output an independent research validation.

## 4. Decision replay

```sh
cargo run --locked -p lt-replay-cli -- journal   --journal /path/to/private/journal.jsonl --evidence /path/to/private/indexes.json   --module /path/to/accepted-component.wasm --out /tmp/sample-decision-replay
```

Retain the original module, host-call results and independent day/order-reference indexes.
Match the module to recorded identity; explicit overrides are divergence experiments, not proof
of the original run. Incomplete/torn journals may verify a prefix but cannot pass complete-day parity.
Journal retrieval from configured storage is also supported; follow the pinned CLI README.

## 5. Supervised paper acceptance

Only after service and broker prerequisites are accepted and the operator authorizes the run:
use exact hashed artifacts on a paper account, compare each day with historical and decision
replay, exercise restart/outage cases and retain differences. Meet Phase 10's at-least-ten-trading-day
acceptance and any sample-specific additional period. Default unsupported capabilities stay off.
Never run two order writers for the same strategy exposure. Native or simulation success does
not qualify protection, partial fills, flatten, margin, WSS or full-session recording.

Every report records exact versions/commands/counts, synthetic versus recorded data, local versus
remote CI, and remaining gates. Public evidence contains no account identifiers, credentials,
private strategies or licensed raw market data.
