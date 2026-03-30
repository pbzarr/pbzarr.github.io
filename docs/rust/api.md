# Rust API Reference

Full API documentation is hosted on docs.rs:

[**pbzarr on docs.rs**](https://docs.rs/pbzarr){ .md-button .md-button--primary }

## Key Types

| Type | Description |
|------|-------------|
| [`PbzStore`](https://docs.rs/pbzarr/latest/pbzarr/struct.PbzStore.html) | Main entry point. Create or open a `.pbz.zarr` store. |
| [`Region`](https://docs.rs/pbzarr/latest/pbzarr/struct.Region.html) | Genomic region with contig, optional start/end. Implements `FromStr`. |
| [`PbzError`](https://docs.rs/pbzarr/latest/pbzarr/enum.PbzError.html) | Error enum covering all failure modes. |

## Error Variants

| Variant | When |
|---------|------|
| `ContigNotFound` | Requested contig doesn't exist in the store |
| `TrackNotFound` | Requested track doesn't exist |
| `ColumnNotFound` | Requested column doesn't exist in the track |
| `InvalidRegion` | Region string couldn't be parsed |
| `InvalidDtype` | Unsupported data type |
| `Store` | Underlying Zarr storage error |
| `Metadata` | Invalid or missing PBZ metadata |

## Dependencies

The crate uses:

- [zarrs](https://crates.io/crates/zarrs) — Zarr v3 implementation
- [ndarray](https://crates.io/crates/ndarray) — N-dimensional arrays
- [thiserror](https://crates.io/crates/thiserror) — Error derive macros
- [serde](https://crates.io/crates/serde) / [serde_json](https://crates.io/crates/serde_json) — Metadata serialization
