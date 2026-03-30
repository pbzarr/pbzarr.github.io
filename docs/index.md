---
hide:
  - navigation
  - toc
---

<div style="text-align: center;" markdown>

![pbzarr banner](assets/pbzarr-banner.svg){ width="600" }

**A Zarr v3 format for per-base resolution genomic data.**

[Get Started (Python)](python/quickstart.md){ .md-button .md-button--primary }
[Format Specification](spec/index.md){ .md-button }

</div>

---

## What is PBZ?

PBZ (Per-Base Zarr) is a convention and domain layer on top of [Zarr v3](https://zarr.dev/) for storing per-base resolution genomic data — depths, methylation levels, quality scores, masks, and more.

It defines a standard store layout and metadata schema so that files written by any PBZ implementation can be read by all others.

## Why PBZ?

Existing formats for per-base genomic data have limitations:

- **bigWig** is read-only after creation, single-track, and lacks modern compression.
- **D4** is fast and compact but single-track and uses a custom binary format.
- **BAM/CRAM** store alignments, not derived signals — extracting depths requires re-computation.

PBZ builds on Zarr v3 to provide:

- **Multi-track stores** — depths, methylation, and masks in one file.
- **Modern compression** — Blosc/Zstd with byte shuffling.
- **Cloud-native** — S3, GCS, and HTTP via Zarr's storage backends.
- **Language-agnostic** — Python and Rust implementations share the same format.
- **Chunked random access** — fast queries over arbitrary genomic regions.

## Quick Example

```python
import numpy as np
import pbzarr

# Create a store with two contigs
store = pbzarr.create_store(
    "sample.pbz.zarr",
    contigs=["chr1", "chr2"],
    contig_lengths={"chr1": 248_956_422, "chr2": 242_193_529},
)

# Add a depth track
track = store.create_track("depth", dtype="i4")

# Write data
track["chr1", :1000] = np.arange(1000, dtype="i4")

# Query a region
data = track.query("chr1:0-1000")
```

## Implementations

| Library | Language | Install | Status |
|---------|----------|---------|--------|
| [pbzarr-py](python/quickstart.md) | Python | `pip install pbzarr` | v0.1.0 |
| [pbzarr-rs](rust/quickstart.md) | Rust | `cargo add pbzarr` | v0.1.0 (partial) |
| pbz (CLI) | Rust | — | Planned |

## Learn More

- [Format Specification](spec/index.md) — the full PBZ format spec
- [Python Quickstart](python/quickstart.md) — install and start using PBZ in Python
- [Rust Quickstart](rust/quickstart.md) — use PBZ from Rust
- [Architecture](design/architecture.md) — design principles and store layout
- [Format Comparison](design/comparison.md) — PBZ vs D4, bigWig, and others
