# Quick start: build and replay the ORB sample

Build the canonical sample from the revision this template supports, then run it with the offline
replay CLI. You need Git, Rust/rustup, a C build toolchain and access to an **isolated, already
prepared** Light-Trader history database. No broker or paper account is needed for this workflow.

## 1. Get the pinned sources and build

Run from the root of `light-trader-sample-strategy`. A fresh directory keeps your normal
Light-Trader checkout and running services untouched:

```sh
LT_SOURCE="$(mktemp -d)"
LT_REV="$(cat light-trader.rev)"
git clone --no-checkout https://github.com/networknt/light-trader.git "$LT_SOURCE"
git -C "$LT_SOURCE" checkout --detach "$LT_REV"
cd "$LT_SOURCE"
rustup target add wasm32-wasip2
cargo test --manifest-path guest/Cargo.toml -p orb-sample --locked
bash scripts/build-guest.sh
sha256sum testdata/wasm/orb_sample.wasm
```

The build script also builds the other upstream guest fixtures. The ORB artifact is
`testdata/wasm/orb_sample.wasm`; retain its hash with your replay results.

## 2. Prepare the offline inputs

Copy `config/replay.example.toml` to `orb-replay.toml` in this fresh checkout. Using the pinned
`bins/lt-replay/README.md` and example schema, set:

- `replay.module` to `testdata/wasm/orb_sample.wasm` and `replay.strategy.strategy` to `orb_sample`.
- Matching account/instrument/calendar/entitlement in `replay.params`, `replay.strategy` and
  `replay.account`; include correct instrument contexts, costs and simulation risk limits.
- Explicit dated sessions and timezone covering the replay period and warm-up. The example's
  short epoch session is a schema example, not an ORB research dataset.
- `database_url_env` to the name of the environment variable supplying your isolated database
  credential. Export it through your local secret mechanism; keep it out of the TOML and shell history.

The database must have the pinned schema and matching minute history, including at least 14
completed prior sessions for ORB warm-up. Use synthetic or appropriately licensed inputs.
The quick start does not import data or run migrations against your normal database. A run with
no eligible opening bars is a plumbing check, not strategy validation.

## 3. Replay and inspect

With that configuration prepared, choose a date range present in your dataset:

```sh
cargo run --locked -p lt-replay-cli -- run \
  --config orb-replay.toml --strategy orb_sample \
  --from 2026-01-02 --to 2026-02-01 --source bars --fill-model next_open \
  --out /tmp/orb-quick-start-report
```

Use a fresh output directory on each run. Dates are exchange-local and the end date is exclusive.
Inspect `summary.json` and `trades.csv`, including failures, the module hash and fill model.
The sample submits after the opening window; the simulation does not fill retroactively at the
opening-window boundary. Follow [testing](testing.md) to compare a reference trade list and
perform decision replay. Paper operation is a later, separately supervised step.
