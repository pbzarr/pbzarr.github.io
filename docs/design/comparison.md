# Format Comparison

How PBZ compares to existing formats for per-base genomic data.

## Overview

| Feature | PBZ | D4 | bigWig | BAM/CRAM |
|---------|-----|----|--------|----------|
| Per-base resolution | Yes | Yes | Yes | Derived |
| Multi-track | Yes | No | No | N/A |
| Multi-column | Yes | No | No | N/A |
| Cloud-native | Yes | No | No | Partial |
| Random access | Yes | Yes | Yes | Yes |
| Compression | Blosc/Zstd | Adaptive | zlib | Various |
| Writable after creation | Yes | Yes | No | No |
| Language support | Python, Rust | C, Python | C, Python | Many |
| Zarr ecosystem | Yes | No | No | No |

## D4

[D4](https://github.com/38/d4-format) is a compact format for per-base quantitative data, optimized for read depth.

**D4 strengths:**

- Mature, battle-tested implementation
- Extremely compact for depth data (adaptive bit-width encoding)
- Fast C implementation with Python bindings

**PBZ advantages:**

- Multi-track stores — depth, methylation, and masks in one file
- Cloud-native via Zarr storage backends (S3, GCS, HTTP)
- Standard format (Zarr v3) rather than custom binary
- Multi-column tracks (e.g., ACGT counts in one track)
- Multiple language implementations that interoperate via spec

## bigWig

[bigWig](https://genome.ucsc.edu/goldenpath/help/bigWig.html) is the standard format for continuous genomic signal tracks.

**bigWig strengths:**

- Universal tool support (IGV, UCSC Genome Browser, deepTools, etc.)
- Mature ecosystem with decades of tooling
- Supports summary statistics at multiple zoom levels

**PBZ advantages:**

- Read-write — bigWig is write-once
- Multi-track stores
- Cloud-native storage
- True per-base resolution without lossy summarization
- Modern compression (Zstd vs zlib)
- Named columns for multi-dimensional data

## BAM/CRAM

BAM and CRAM store aligned sequencing reads, not derived signals.

**When to use BAM/CRAM:** You need actual alignments — variant calling, structural variant detection, visualization of individual reads.

**When to use PBZ:** You've already computed a per-base signal (depth, methylation, quality) and want to store and query it efficiently without re-processing alignments.

PBZ stores are typically 10-100x smaller than the BAM files they derive from, since they store only the computed signal rather than full read-level data.

## Custom HDF5

Some projects store genomic data in HDF5 with ad-hoc schemas.

**PBZ advantages over ad-hoc HDF5:**

- Standardized schema — no reinventing the store layout per project
- Zarr v3 is cloud-native by design (HDF5 cloud support is limited)
- Cross-language interop guaranteed by the spec
- Zarr's chunk-level access is well-suited to genomic range queries
- Growing ecosystem of Zarr-aware tools
