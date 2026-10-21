# Writing a strategy

Create a Rust component using `light-trader/guest/lt-strategy-guest` and the
[compatibility baseline](compatibility.md). Start with the [ORB quick start](quick-start.md) to
build an existing sample, then implement your own strict parameter schema and callbacks.

## Build boundary

A sample crate uses `cdylib` (component) and optionally `rlib` (native tests). Pin the SDK to an
accepted full Git revision and retain the lockfile/toolchain. A local sibling SDK path is useful
for development but must be replaced or resolved in clean-checkout CI. Verify the SDK's bundled
WIT files are available through the chosen dependency mechanism before claiming S1 complete.

For the current implemented examples, these commands run from the **Light-Trader root**, not
this documentation-only repository:

```sh
rustup target add wasm32-wasip2
cargo test --manifest-path guest/Cargo.toml -p orb-sample -p sma-cross --locked
bash scripts/build-guest.sh
sha256sum testdata/wasm/orb_sample.wasm testdata/wasm/sma_cross.wasm
```

They are reproducible build/test instructions, not checks executed by this documentation task.
Use the pinned checkout's required Rust toolchain and component inspection scripts.

## Lifecycle

```rust
use lt_strategy_guest::{export_strategy, prelude::*};

#[derive(serde::Deserialize)]
#[serde(deny_unknown_fields)]
struct Params {
    account: String,
    inst: String,
    entitlement: String,
}

pub struct Observe;
impl Strategy for Observe {
    fn init(params: &str) -> Result<(Self, StrategySpec), String> {
        let p: Params = serde_json::from_str(params).map_err(|e| e.to_string())?;
        if p.account.trim().is_empty() || p.entitlement.trim().is_empty() {
            return Err("account and entitlement must be nonempty".into());
        }
        InstrumentRef::parse(&p.inst)?;
        let spec = StrategySpec {
            account: p.account,
            instruments: vec![p.inst],
            entitlement: p.entitlement,
            data: vec![],
            timers: vec![],
        };
        Ok((Self, spec))
    }
    fn on_start(&mut self, host: &Host) -> Result<(), String> {
        host.log(LogLevel::Info, r#"{"event":"started"}"#);
        Ok(())
    }
}
export_strategy!(Observe);
```

This minimal lifecycle illustration accepts only `account`, `inst` and `entitlement`, constructs
its own spec and sends no orders. Use the actual ORB/SMA source for strict
parameter parsing and explicit `TimerSpec`, `BarQuery` and `OrderIntent` construction. A strategy
spec declares account alias, instruments, data kinds, entitlement and calendar-relative timers.
The host validates it against deployment policy. Other callbacks default to no action.

Use `Host::now`, `delivery`, `session`, `bars`, `instrument`, `position` and `open_orders` rather
than direct I/O. Use `load_state_json`, `save_state_json` and `log_json` for typed JSON objects;
handle their errors. Admission errors must not be swallowed or converted to assumed fills.

## Tests and promotion

Native tests opt into `lt-strategy-guest`'s `testing` feature and pass a scripted `FakeHost` into
callbacks. Then build and test the actual component with the host and engine. Keep instruction,
wall-time, memory and host-call limits explicit; no native test proves WASM resource enforcement.

Promote by exact module SHA-256 and record SDK/WIT/host, parameter/state versions and evidence.
Separate replay configuration from paper trade-server configuration. Do not use a missing hash
as an upgrade convenience or copy a new module over the accepted artifact while it is running.
The default upgrade is a reviewed state-compatible stop/start with reconciliation; hot swapping
requires its own implemented and qualified path.

Other languages can target the WIT component world, but their toolchains/imports and determinism
need separate qualification. This repository's initial supported implementation language is Rust.
