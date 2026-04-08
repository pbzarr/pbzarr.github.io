# PBZ (Per-Base Zarr) Specification

***Version 0.1 — DRAFT***

This document is a technical specification for PBZ, a means of encoding per-base resolution genomic data in chunked form using the Zarr format. Per-base data includes, for example, read depth, methylation levels, accessibility scores, and boolean masks across multiple samples or other column types.

This specification depends on definitions and terminology from the [Zarr v3 core specification](https://zarr-specs.readthedocs.io/en/latest/v3/core/v3.0.html).

PBZ defines a layout for per-base genomic data within a Zarr v3 store. Array storage, compression, chunking, and I/O are delegated to the Zarr implementation.

## Conventions

The key words "MUST", "MUST NOT", "SHOULD", "SHOULD NOT", and "MAY" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

## File extension

PBZ stores use the `.pbz.zarr` extension.

## Overall structure

A PBZ store is composed of a top-level Zarr group containing store-level metadata as group attributes, contig information stored as Zarr arrays, and a `tracks` subgroup containing one subgroup per track.

```
<store>.pbz.zarr/
├── zarr.json
├── contigs
├── contig_lengths
└── tracks/
    ├── <track_name>/
    │   ├── zarr.json
    │   ├── columns
    │   ├── <contig_name>
    │   └── ...
    └── <group>/<track_name>/
        └── ...
```

Tracks MAY be nested in subgroups beneath `tracks` (for example, `tracks/masks/callable`).

## PBZ Zarr group attributes

The root Zarr group contains the following mandatory attribute namespace under the key `perbase_zarr`:

| Key       | Value     |
|-----------|-----------|
| `version` | `"0.1"`   |

## Contig arrays

Contig names and lengths are stored as one-dimensional Zarr arrays at the root of the store.

| Path             | Shape            | Dimension names | Dtype   | Description                  |
|------------------|------------------|-----------------|---------|------------------------------|
| `contigs`        | `(num_contigs,)` | `[contigs]`     | `str`   | Contig names                 |
| `contig_lengths` | `(num_contigs,)` | `[contigs]`     | `int64` | Contig lengths in base pairs |

The order of contigs MUST be consistent between these two arrays.

## Tracks

A track is a named Zarr group containing per-base data for all contigs. Tracks are stored under the `tracks` group and MAY be nested in subgroups.

### Track group attributes

Each track group contains a mandatory attribute namespace under the key `perbase_zarr_track` with the following keys:

| Key                 | Type    | Required                          | Description                                                                                 |
|---------------------|---------|-----------------------------------|---------------------------------------------------------------------------------------------|
| `dtype`             | string  | Yes                               | One of: `uint8`, `uint16`, `uint32`, `int8`, `int16`, `int32`, `float32`, `float64`, `bool` |
| `chunk_size`        | integer | Yes                               | Chunk size along the position axis, in base pairs                                           |
| `column_chunk_size` | integer | If `has_columns`                  | Chunk size along the column axis                                                            |
| `has_columns`       | boolean | Yes                               | Whether the track has a column dimension                                                    |
| `shard_size`        | integer | No                                | Shard size along the position axis, in base pairs. If absent, the track is not sharded.    |
| `shard_column_size` | integer | If `shard_size` and `has_columns` | Shard size along the column axis                                                            |
| `description`       | string  | No                                | Human-readable description                                                                  |
| `source`            | string  | No                                | Tool and version that created the track                                                     |

Tools MAY include additional keys in the `perbase_zarr_track` namespace. To avoid collisions, tool-specific metadata SHOULD be placed under a single key matching the tool name. Implementations MUST preserve unrecognized keys when reading and writing track metadata.

### Columns array

If `has_columns` is `true`, the track group MUST contain a `columns` array:

| Path      | Shape            | Dimension names | Dtype | Description        |
|-----------|------------------|-----------------|-------|--------------------|
| `columns` | `(num_columns,)` | `[columns]`     | `str` | Column identifiers |

### Data arrays

Each track contains one Zarr array per contig, at a path matching the contig name. Shape and chunks depend on `has_columns`:

| `has_columns` | Shape                          | Chunks                            | Dimension names      |
|---------------|--------------------------------|-----------------------------------|----------------------|
| `true`        | `(contig_length, num_columns)` | `(chunk_size, column_chunk_size)` | `[position, column]` |
| `false`       | `(contig_length,)`             | `(chunk_size,)`                   | `[position]`         |

Dtype is as specified in the track attributes. Dimension names are stored following [Xarray conventions](https://docs.xarray.dev/en/stable/internals/zarr-encoding-spec.html) as the `_ARRAY_DIMENSIONS` attribute on each data array.

## Chunking and sharding

Implementations SHOULD use the following default chunk sizes when creating tracks:

| Parameter           | Default   |
|---------------------|-----------|
| `chunk_size`        | 1,000,000 |
| `column_chunk_size` | 16        |

(*Implementation note: Large position chunks yield higher compression ratios and efficient sequential access. Small column chunks reduce the cost of appending columns, since only edge chunks are rewritten, and enable parallel decompression for cross-column queries.*)

(*Implementation note: For tracks with many columns, `column_chunk_size` of 32–64 may perform better. For tracks with frequent column additions, smaller values (8–16) reduce write amplification. For contigs shorter than `chunk_size`, implementations may set `chunk_size` to the contig length.*)

Sharding is OPTIONAL. If `shard_size` is present in the track attributes, the track is sharded and implementations MUST create the underlying Zarr arrays with the corresponding shard shape. For a track with `has_columns` set to `true`, the shard shape is `(shard_size, shard_column_size)`; otherwise it is `(shard_size,)`.

(*Implementation note: Without sharding, each chunk is stored as a separate object. For a 250 Mbp contig with a 1,000,000 bp `chunk_size`, this produces 250 objects per contig per track. Sharding reduces object count at the cost of increased write amplification, since writing a single chunk may require rewriting the containing shard. Sharding is recommended for stores hosted on object storage or stores with many tracks and contigs.*)

## Compression

PBZ does not mandate a compression codec. Implementations SHOULD default to Zstandard compression at level 5 with byte shuffle, via the Blosc codec, and MAY use any Zarr-compatible codec. Compression configuration is handled by the underlying Zarr implementation.

## Coordinate system

Positions are 0-based and half-open, consistent with the BED format. Position `i` in a data array corresponds to reference base pair `[i, i+1)`. Contig names MUST match the reference genome exactly.

## Missing data

PBZ uses the Zarr `fill_value` mechanism to represent missing data. The `fill_value` is set at array creation time and is returned by the Zarr implementation for any chunk that has never been written.

Implementations SHOULD use the following default fill values:

| Dtype category | Default fill value |
|----------------|--------------------|
| Integer        | `0`                |
| Float          | `NaN`              |
| Bool           | `false`            |

Implementations MAY allow users to override the fill value at track creation time.

PBZ does not define sentinel values or a missing-data mask convention. Applications requiring a distinction between "zero" and "no data" SHOULD use a float dtype, where `NaN` serves as a missing indicator, or manage sentinel conventions at the application layer.