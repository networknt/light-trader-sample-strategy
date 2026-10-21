# Compatibility baseline and sample ownership

[`light-trader.rev`](https://github.com/networknt/light-trader-sample-strategy/blob/master/light-trader.rev)
at the root of this repository is the single Light-Trader source pin for this template. The
2026-10-21 review checked the SDK, WIT, examples and replay CLI against that revision. Read the file
in commands and automation rather than duplicating its value in documentation or build scripts.

## Ownership decision

**Light-Trader owns the canonical ORB and SMA samples**, under `guest/examples/orb-sample` and
`guest/examples/sma-cross`. This repository pins that checkout and supplies teaching material,
configuration/fixture templates and build/replay orchestration. It does not maintain copies of
those strategy implementations. Fix sample logic/lifecycle defects upstream, then advance the pin
in a reviewed change with new validation evidence.

Users create their own named component crates for custom strategies; that is a deliberate new
strategy, not a second maintained ORB/SMA reference. The same applies to private strategies.

## Sources at the pinned checkout

| Contract | Location |
|---|---|
| WIT | `wit/strategy.wit`, package `light-trader:strategy@0.1.0` |
| Rust SDK / fake host | `guest/lt-strategy-guest`, native test feature `testing` |
| Canonical examples | `guest/examples/orb-sample`, `guest/examples/sma-cross` |
| Guest build | `scripts/build-guest.sh` and `guest/Cargo.toml` |
| Replay CLI | Cargo package `lt-replay-cli`, commands `run`, `compare`, `journal` |
| Offline config | `config/replay.example.toml` and `bins/lt-replay/README.md` |

On a pin update, review WIT/SDK/config/CLI changes, build actual components, run their native and
host/replay tests, compare synthetic outcomes and refresh artifact hashes. Record toolchain,
lockfile and host compatibility in build manifests. A WIT version string is not a substitute
for that review. Paper acceptance and published image availability are separate checks.

The broader [plugin rationale and concurrency contract](https://github.com/networknt/light-trader/blob/master/docs/src/design/plugins.md)
is maintained in Light-Trader; this book explains how to consume it, not a fork of that design.
