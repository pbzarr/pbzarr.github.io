# Python API Reference

The `pbzarr` package exposes a small surface: a store class, a track class,
region helpers, a backend enum, and an exception hierarchy. Most users will
only need [`create_store`][pbzarr.create_store], [`open_store`][pbzarr.open_store],
and the `[]` indexing on `PbzStore` and `Track`.

## Stores

A PBZ store is the top-level handle. It wraps a zarr v3 root group and
exposes contigs, tracks, and metadata. Create one with
[`create_store`][pbzarr.create_store] or open an existing one with
[`open_store`][pbzarr.open_store] — the aliases [`pbzarr.create`][pbzarr.create_store]
and [`pbzarr.open`][pbzarr.open_store] are also available.

::: pbzarr.PbzStore

::: pbzarr.create_store

::: pbzarr.open_store

---

## Tracks

A track is a named group of per-base data arrays under `/tracks/{name}/`.
Tracks are created via [`PbzStore.create_track`][pbzarr.PbzStore.create_track]
and accessed by indexing a store: `store["depths"]`. Both `query()` and
`[]`-style indexing are supported for reads; `[]` indexing is also supported
for writes.

::: pbzarr.Track

---

## Regions

Genomic regions are parsed into a [`Region`][pbzarr.Region] dataclass in
0-based half-open coordinates. You rarely need to construct one directly —
[`parse_region`][pbzarr.parse_region] handles both string (`"chr1:1000-2000"`)
and tuple (`("chr1", 1000, 2000)`) forms, and `Track.query()` calls it for you.

::: pbzarr.Region

::: pbzarr.parse_region

---

## Backends

PBZ supports two array backends for data retrieval: eager numpy arrays and
lazy dask arrays. The backend is chosen when opening a store and inherited
by all tracks from that store. See [Backends](backends.md) for guidance on
when to use each.

::: pbzarr.Backend

---

## Exceptions

All PBZ exceptions inherit from [`PbzError`][pbzarr.PbzError], so you can
catch the base class to handle any pbzarr-specific failure.

::: pbzarr.PbzError

::: pbzarr.ContigNotFoundError

::: pbzarr.TrackNotFoundError

::: pbzarr.InvalidRegionError

::: pbzarr.ColumnNotFoundError