# light-trader-sample-strategy
Public WASM strategy examples and deployment-template design for Light-Trader.
ORB/SMA implementations remain canonical in Light-Trader; this template pins their revision in
`light-trader.rev`. Start with the [build and replay quick start](docs/src/samples/quick-start.md).

Start with [architecture](docs/src/samples/architecture.md) and
[implementation milestones](docs/src/samples/implementation.md).

## Documentation

The project documentation uses [mdBook](https://rust-lang.github.io/mdBook/).
See [docs/README.md](docs/README.md) for installation and preview instructions.

```sh
mdbook build docs
mdbook serve docs --hostname 127.0.0.1 --port 3000
```
