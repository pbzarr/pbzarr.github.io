# Rust Quickstart

## Installation

Add `pbzarr` to your project:

```bash
cargo add pbzarr
```

Or add it to `Cargo.toml` manually:

```toml
[dependencies]
pbzarr = "0.1"
```

## Creating a Store

```rust
use pbzarr::PbzStore;
use std::collections::HashMap;
use std::path::Path;

fn main() -> pbzarr::Result<()> {
    let contigs = vec!["chr1".to_string(), "chr2".to_string()];
    let contig_lengths: HashMap<String, u64> = [
        ("chr1".to_string(), 248_956_422),
        ("chr2".to_string(), 242_193_529),
    ]
    .into();

    let store = PbzStore::create(
        Path::new("sample.pbz.zarr"),
        &contigs,
        &contig_lengths,
    )?;

    println!("Created store at {:?}", store.path());
    Ok(())
}
```

## Opening a Store

```rust
use pbzarr::PbzStore;
use std::path::Path;

fn main() -> pbzarr::Result<()> {
    let store = PbzStore::open(Path::new("sample.pbz.zarr"))?;

    println!("Contigs: {:?}", store.contigs());
    println!("Contig lengths: {:?}", store.contig_lengths());

    // Get a specific contig's length
    let len = store.contig_length("chr1")?;
    println!("chr1 length: {len}");

    Ok(())
}
```

## Parsing Regions

The `Region` type represents a genomic interval. It implements `FromStr` for parsing from strings:

```rust
use pbzarr::Region;

fn main() -> pbzarr::Result<()> {
    // Full region
    let region: Region = "chr1:1000-2000".parse()?;
    assert_eq!(region.contig, "chr1");
    assert_eq!(region.start, Some(1000));
    assert_eq!(region.end, Some(2000));

    // Contig only (whole contig)
    let region: Region = "chr1".parse()?;
    assert_eq!(region.start, None);
    assert_eq!(region.end, None);

    // Start only (to end of contig)
    let region: Region = "chr1:5000".parse()?;
    assert_eq!(region.start, Some(5000));
    assert_eq!(region.end, None);

    // Commas in numbers are stripped
    let region: Region = "chr1:1,000-2,000".parse()?;
    assert_eq!(region.start, Some(1000));

    Ok(())
}
```

## Current Status

The Rust library provides store creation/opening and region parsing. Track-level I/O is under active development.

| Feature | Status |
|---------|--------|
| Store create/open | Done |
| Region parsing | Done |
| Error types | Done |
| Track metadata | In progress |
| Track read/write | Planned |
| Sharding | Planned |

For the full API, see the [API Reference](api.md).
