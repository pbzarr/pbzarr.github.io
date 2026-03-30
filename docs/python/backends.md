# Backends

pbzarr supports two array backends: **NumPy** (default) and **Dask**.

## NumPy (default)

Returns standard `numpy.ndarray` objects. Data is loaded eagerly into memory.

```python
import pbzarr

store = pbzarr.open_store("sample.pbz.zarr")  # backend="numpy" is default
data = store["depth"].query("chr1:0-1000")
type(data)  # numpy.ndarray
```

Best for:

- Small to medium datasets that fit in memory
- Interactive exploration and debugging
- Pipelines that use NumPy, pandas, or scikit-learn

## Dask

Returns `dask.array.Array` objects. Data is loaded lazily — computation happens only when you call `.compute()`.

```python
store = pbzarr.open_store("sample.pbz.zarr", backend="dask")
data = store["depth"].query("chr1")

# No data loaded yet
type(data)  # dask.array.Array
print(data.shape)  # shape is known without loading data

# Load data when ready
result = data.compute()
```

Best for:

- Whole-genome queries that exceed available memory
- Parallel computation across cores or clusters
- Lazy pipelines where you want to defer computation

### Installation

Dask is an optional dependency:

```bash
pip install "pbzarr[dask]"
```

If you open a store with `backend="dask"` without Dask installed, you'll get a clear error message telling you to install it.

## Choosing a Backend

The backend is set when opening a store and applies to all queries from that store:

```python
# NumPy — eager, in-memory
store = pbzarr.open_store("data.pbz.zarr")
store = pbzarr.open_store("data.pbz.zarr", backend="numpy")

# Dask — lazy, out-of-core
store = pbzarr.open_store("data.pbz.zarr", backend="dask")
```

| | NumPy | Dask |
|---|---|---|
| Loading | Eager (immediate) | Lazy (on `.compute()`) |
| Memory | Must fit in RAM | Can exceed RAM |
| Parallelism | Single-threaded | Multi-core / distributed |
| Dependencies | Included | Optional (`pbzarr[dask]`) |
| Best for | Interactive, small data | Batch, large data |

## Escape Hatch

Both backends return arrays from the same underlying Zarr data. If you need direct access to the Zarr array (e.g., for chunk-level operations or advanced codec control):

```python
track = store["depth"]
zarr_array = track.zarr_array("chr1")  # raw zarr.Array, independent of backend
```
