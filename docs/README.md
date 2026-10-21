# Light-Trader Sample Strategy documentation

This mdBook uses the configuration and Mermaid diagram support from Light-Fabric.

Install the documentation tools (requires Rust and Cargo):

```sh
cargo install mdbook --locked
cargo install mdbook-mermaid --locked
```

From the repository root, build the book:

```sh
mdbook build docs
```

Preview it at <http://localhost:3000>:

```sh
mdbook serve docs --hostname 127.0.0.1 --port 3000
```

Edit Markdown files in `docs/src/` and register new chapters in
`docs/src/SUMMARY.md`. Generated HTML goes to `docs/book/` and is ignored by Git.

Mermaid diagrams can be written in fenced code blocks marked `mermaid`.
