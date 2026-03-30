# Python Quickstart

## Installation

Install from PyPI:

```bash
pip install pbzarr
```

For Dask support (lazy, out-of-core computation):

```bash
pip install "pbzarr[dask]"
```

## Creating a Store

A PBZ store holds per-base data for a set of contigs (chromosomes). Create one by specifying the contig names and their lengths:

```python
import pbzarr

store = pbzarr.create_store(
    "sample.pbz.zarr",
    contigs=["chr1", "chr2", "chrX"],
    contig_lengths={
        "chr1": 248_956_422,
        "chr2": 242_193_529,
        "chrX": 156_040_895,
    },
)
```

The store is a directory on disk (or a cloud object prefix) with the `.pbz.zarr` extension.

## Adding Tracks

Tracks hold per-base data arrays. Each track has a data type and optional column names:

```python
import numpy as np

# Single-value track (one value per base position)
depth = store.create_track("depth", dtype="i4")

# Multi-column track (multiple values per base position)
counts = store.create_track(
    "base_counts",
    dtype="i4",
    columns=["A", "C", "G", "T"],
)
```

Track creation accepts optional parameters:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `dtype` | *(required)* | NumPy dtype string (`"i4"`, `"f4"`, etc.) |
| `columns` | `None` | Column names for multi-column tracks |
| `chunk_size` | `1_000_000` | Positions per chunk |
| `fill_value` | `0` | Default value for unwritten positions |
| `description` | `None` | Human-readable description |
| `source` | `None` | How the data was generated |

## Writing Data

Write data using slice syntax:

```python
# Write 1000 values to the start of chr1
depth["chr1", :1000] = np.arange(1000, dtype="i4")

# Write to a specific range
depth["chr1", 5000:6000] = np.ones(1000, dtype="i4")

# Write to a specific column
counts["chr1", :1000, "A"] = np.ones(1000, dtype="i4")
```

## Querying Data

Query data using genomic region strings:

```python
# Query a region (0-based, half-open)
data = depth.query("chr1:0-1000")

# Query an entire contig
data = depth.query("chr1")

# Query specific columns
a_counts = counts.query("chr1:0-1000", columns=["A", "C"])

# One-based coordinates (converts automatically)
data = depth.query("chr1:1-1000", one_based=True)
```

Regions use 0-based, half-open coordinates by default (BED convention). Pass `one_based=True` when working with 1-based formats like VCF or SAM.

## Opening an Existing Store

```python
store = pbzarr.open_store("sample.pbz.zarr")

# List available tracks
print(store.tracks())  # ["depth", "base_counts"]

# Access a track by name
depth = store["depth"]

# Inspect track metadata
print(depth.dtype)       # "int32"
print(depth.columns)     # None (single-value track)
print(depth.description) # None or a string

# Query
data = depth.query("chr1:0-1000")
```

## Store Metadata

```python
# Available contigs
print(store.contigs)         # ["chr1", "chr2", "chrX"]
print(store.contig_lengths)  # {"chr1": 248956422, ...}
```

## Using Dask

For large datasets, open with the Dask backend for lazy computation:

```python
store = pbzarr.open_store("sample.pbz.zarr", backend="dask")
depth = store["depth"]

# Returns a dask array — no data loaded yet
data = depth.query("chr1")

# Compute when ready
result = data.compute()
```

See [Backends](backends.md) for details on when to use NumPy vs Dask.

## Escape Hatch

If the PBZ API doesn't cover your use case, access the underlying Zarr objects directly:

```python
# Raw zarr.Array for a contig
zarr_array = depth.zarr_array("chr1")

# Raw zarr.Group (store root)
root_group = store.root
```
