# Sample strategies

This public repository is the planned teaching and integration template for Light-Trader WASM
strategies. Keep strategy development independent of the engine's implementation and deployment.

> **Status:** this repository contains documentation and a source revision pin. ORB/SMA source
> remains canonical in Light-Trader; template automation and Compose profiles are planned.
> Builds and replay can use the upstream checkout now. Paper deployment requires separate acceptance.

- [Quick start: build and replay ORB](quick-start.md).
- [Compatibility and ownership](compatibility.md): one source pin and upstream sample ownership.
- [Architecture and contracts](architecture.md): repository boundaries, SDK, time, state and artifacts.
- [Implementation milestones](implementation.md): S0–S6 and acceptance evidence.
- [Writing a strategy](writing.md): current API and development workflow.
- [Opening range breakout](orb.md): exact implemented reference behavior and hardening requirements.
- [Moving-average crossover](sma.md): the one-share daily crossover reference.
- [Testing](testing.md): native, compiled-component, historical, decision and paper evidence.
- [Stack design](compose.md): isolated offline default and separately enabled paper deployment.

The samples illustrate the contracts, not investment recommendations. They do not contain
private strategy filters, account details, research outputs or licensed market-data exports.
