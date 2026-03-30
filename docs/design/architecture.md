# Architecture

## Design Principles

### Convention Over Configuration

PBZ is a **convention layer** on top of Zarr v3. It defines a standard layout and metadata schema but delegates all array storage, compression, and I/O to the underlying Zarr library.

This means:

- No custom binary format to maintain or evolve
- Automatic support for new Zarr storage backends and codecs
- Any Zarr-compatible tool can inspect PBZ files directly

### Escape Hatches

Every PBZ implementation exposes the underlying Zarr objects. If the PBZ API doesn't cover your use case, drop down to Zarr:

=== "Python"

    ```python
    track = store["depth"]
    zarr_array = track.zarr_array("chr1")  # raw zarr.Array
    root_group = store.root                # raw zarr.Group
    ```

=== "Rust"

    ```rust
    let storage = store.storage();  // raw Zarr storage handle
    ```

### Cross-Language Interoperability

Files written by any PBZ implementation must be readable by all others. The [format specification](../spec/index.md) defines canonical metadata keys and store layout to guarantee this.

## Store Layout

A PBZ store is a Zarr v3 group with this structure:

```
sample.pbz.zarr/
├── zarr.json                       # Zarr v3 root group metadata
├── contigs/                        # 1-D string array
│   └── ...                         # ["chr1", "chr2", ...]
├── contig_lengths/                  # 1-D uint64 array
│   └── ...                         # [248956422, 242193529, ...]
└── tracks/
    ├── depth/                      # A single-value track
    │   ├── zarr.json               # Track group + perbase_zarr_track metadata
    │   ├── chr1/                   # 1-D array, one element per base
    │   │   └── ...
    │   └── chr2/
    │       └── ...
    └── base_counts/                # A multi-column track
        ├── zarr.json
        ├── chr1/                   # 2-D array, shape (contig_length, num_columns)
        │   └── ...
        └── chr2/
            └── ...
```

### Root Metadata

The root group carries a `perbase_zarr` attribute identifying the store and its spec version:

```json
{
  "perbase_zarr": {
    "version": "0.1"
  }
}
```

### Track Metadata

Each track group carries a `perbase_zarr_track` attribute with track configuration:

```json
{
  "perbase_zarr_track": {
    "dtype": "int32",
    "chunk_size": 1000000,
    "column_chunk_size": 16,
    "has_columns": false,
    "fill_value": 0,
    "description": "Read depth from samtools depth",
    "source": "samtools depth -a sample.bam"
  }
}
```

For multi-column tracks, `has_columns` is `true` and the metadata includes column names:

```json
{
  "perbase_zarr_track": {
    "dtype": "int32",
    "chunk_size": 1000000,
    "column_chunk_size": 16,
    "has_columns": true,
    "columns": ["A", "C", "G", "T"],
    "fill_value": 0
  }
}
```

## Coordinate System

PBZ uses **0-based, half-open coordinates** everywhere internally. This is the BED convention:

| Convention | First base | Region `[0, 1000)` | Length |
|-----------|-----------|-------------------|--------|
| 0-based, half-open (PBZ) | 0 | Bases 0–999 | 1000 |
| 1-based, closed (VCF/SAM) | 1 | Bases 1–1000 | 1000 |

Conversion from 1-based formats happens at the API boundary via explicit flags — PBZ never guesses:

=== "Python"

    ```python
    data = track.query("chr1:1-1000", one_based=True)
    ```

=== "Rust"

    ```rust
    // Region parsing always uses 0-based coordinates.
    // Convert before parsing if your input is 1-based.
    ```

## Chunking and Compression

### Default Configuration

| Parameter | Default | Description |
|-----------|---------|-------------|
| Chunk size | 1,000,000 | Positions per chunk (1 Mbp) |
| Column chunk size | 16 | Columns per chunk |
| Compression | Blosc(Zstd, 5, byte shuffle) | Codec chain |

These defaults balance random access performance with compression ratio for typical genomic data.

### Sharding

PBZ supports Zarr v3 sharding for stores with many small contigs. Sharding groups multiple chunks into a single storage object, reducing the number of files on disk or objects in cloud storage.

## Data Model

### Single-Value Tracks

One value per base position. The data array for each contig is 1-D with shape `(contig_length,)`.

```
chr1: [0, 0, 1, 5, 12, 30, 45, ...]  →  248,956,422 values
```

Examples: read depth, GC content, mappability scores, binary masks.

### Multi-Column Tracks

Multiple named values per base position. The data array for each contig is 2-D with shape `(contig_length, num_columns)`.

```
chr1 (shape: 248956422 × 4):
  A: [0, 0, 1, 0, ...]
  C: [0, 1, 0, 0, ...]
  G: [1, 0, 0, 1, ...]
  T: [0, 0, 0, 0, ...]
```

Examples: per-base nucleotide counts, methylation (beta + coverage), allele-specific signals.
