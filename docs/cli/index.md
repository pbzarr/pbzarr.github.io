# Command-Line Interface

!!! note "Coming Soon"
    The `pbz` CLI is under development. It will provide command-line tools for creating, inspecting, and converting PBZ files.

## Planned Features

- **`pbz create`** — Create PBZ stores from BAM, D4, or bigWig files
- **`pbz view`** — Query and display per-base data from a region
- **`pbz info`** — Show store metadata (contigs, tracks, compression)
- **`pbz convert`** — Convert between PBZ and other formats

## Repository

The CLI will be a Rust binary in the [`pbz`](https://github.com/pbzarr/pbz) repository, built on top of the [`pbzarr`](https://crates.io/crates/pbzarr) library crate.
