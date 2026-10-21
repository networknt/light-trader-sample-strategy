# Contributing to the documentation

Install [Rust and Cargo](https://www.rust-lang.org/tools/install), then install
mdBook and its Mermaid preprocessor:

```sh
cargo install mdbook --locked
cargo install mdbook-mermaid --locked
```

Run these commands from the repository root:

```sh
# Generate HTML in docs/book/.
mdbook build docs

# Preview the book at http://localhost:3000.
mdbook serve docs --hostname 127.0.0.1 --port 3000
```

Create chapters as Markdown files under `docs/src/` and add their links to
`docs/src/SUMMARY.md`. Rebuild the book to check each new page.

## Diagrams

Use Mermaid fenced code blocks for diagrams:

```mermaid
flowchart LR
    Markdown[Markdown chapters] --> mdBook
    mdBook --> HTML[HTML documentation]
```
